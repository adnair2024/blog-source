+++
date = '2026-09-26T10:59:30-06:00'
draft = false
title = 'A quick Neovim Plugin that made writing code and blog articles easier for me'
type = 'post'
tags = ['neovim', 'coding', 'markdown', 'notes']
categories = ['Neovim']
+++

## Table of Contents

* [A quick Neovim Plugin that made writing code and blog articles easier for me](#a-quick-neovim-plugin-that-made-writing-code-and-blog-articles-easier-for-me)
    * [Why I needed temp.nvim](#why-i-needed-tempnvim)
    * [Planning temp.nvim](#planning-tempnvim)
    * [The Architecture](#the-architecture)
      * [templates.lua - The Template Registry](#templateslua---the-template-registry)
      * [templates.lua - The rendering function](#templateslua---the-rendering-function)
      * [init.lua - Path detection](#initlua---path-detection)
      * [init.lua - Input handling](#initlua---input-handling)
      * [init.lua - User command setup](#initlua---user-command-setup)
    * [Conclusion](#conclusion)

### Why I needed temp.nvim

Recently, I have been getting less busy with my new job. I enjoy it, but I also wanted to 
continue writing articles about little things I like. Something that has been subconsciously
turning me off to writing new articles is the process of making them with Hugo; Hugo is 
a great static site generator for blogs, but you have to create the front matter or edit it
significantly even if you copy and paste from an older article you wrote. This got me 
thinking about docstrings, and I started thinking about an my first neovim config (a 
ridiculously long init.lua file that had plugins, keymaps and vim commands) had keymaps
for making docstrings for my .py and .js files. I decided to make a neovim plugin to solve this
and figured other people may enjoy either using it or seeing how I made it. Many people 
use Hugo and billions of people write code, meaning many people could benefit from this 
plugin. I decided to name it temp.nvim, and I started planning

### Planning temp.nvim

My main goals for temp.nvim were handling two things seamlessly:
-   Smart template injection: Automatically looking at the filetype and directory I'm
    located in and determining which template to use

-   Interactive titles: Prompting me to type out a custom title in a floating input box
    instead of hardcoding.

For my blog specifically, I needed to make it inject Hugo TOML front matter with automatic
timestamp generation and premade tags and categories so I can save my brainpower for the 
actual article.

### The Architecture

I really wanted something clean and streamlined, so I split this into two files:
-   templates.lua - This file holds a registry of template strings mapped to keys like
    `markdown`, `hugo_post`, and `python` and handles string conversion for placeholders 
    like `{{title}}`, `{{date}}`, and `{{timestamp}}`.
-   init.lua - Does most of the heavy lifting, like registering the Neovim user command
    `:Temp`, sets up the path detection (for my usecase, it checks to see if I'm in the 
    `content/post/` directory) to see which header it should use.

Starting off with templates.lua:

#### templates.lua - The Template Registry

```lua

local M = {}

M.registry = {
    markdown = [[
---
title: "{{title}}"
date: {{date}}
draft: true
tags: []
---

# {{title}}
]],
    hugo_post = [[
+++
date = '{{timestamp}}'
draft = false
title = '{{title}}'
type = 'post'
tags = []
categories = []
+++

# {{title}}
]],
    python = [[
#!/usr/bin/env python3
"""
Author: Ashwin Nair
Created: {{date}}
Description: {{title}}
"""

def main():
    pass

if __name__ == "__main__":
    main()
]]

}```

The templates.lua file is just meant to be a data layer used to store templates and 
a function to render those templates. We're storing our templates in a lookup table 
called `M.registry`. Each key maps to a string with placeholders configured like 
`{{title}}`, `{{date}}`, and `{{timestamp}}`.

#### templates.lua - The rendering function

```lua
function M.render(template_key, data)
    local template = M.registry[template_key]
    if not template then return nil end

    for key, value in pairs(data) do
        template = template:gsub("{{" .. key .. "}}", value)
    end

    return vim.split(template, "\n")
end

return M
```

This function looks up the template by the key provided, loops through the data table, and 
replaces each instance of `{{key}}` with its corresponding value using string pattern matching
(we use `:gsub()` for this). Then,it splits the resulting string by newlines so Neovim's
buffer API can consume it as a list of lines.

Moving into init.lua, we are heading into the core logic, including the UI prompt.

#### init.lua - Path detection

```lua
local templates = require("temp.templates")
local M = {}

local function insert_template(template_type)
    local ft = vim.bo.filetype
    local chosen_template = template_type
    
    if not chosen_template or chosen_template == "" then
        local current_file = vim.fn.expand("%:p")
        if current_file:match("content/post") or current_file:match("blog") then
            chosen_template = "hugo_post"
        else
            chosen_template = ft ~= "" and ft or "markdown"
        end
    end

    local default_filename = vim.fn.expand("%:t:r")
    local default_title = default_filename ~= "" and default_filename:gsub("-", " "):gsub("^%l", string.upper) or "Untitled"
```

When a user runs a command, we first check if they passed an argument. If they didn't,
we look at the current file path using `vim.fn.expand("%:p")`. If you're working inside a 
blog directory, it automatically defaults to `hugo_post`. I think this is important because if you
wanted different directories to detect, you could change `content/post` to whatever other
directory you want; for example it could be `notes/english` if you want to automatically
generate a header for your notes in markdown. You could even go as far as making different
headers for different subjects.

#### init.lua - Input handling

```lua
vim.ui.input({
        prompt = "Template Title (" .. chosen_template .. "): ",
        default = default_title,
    }, function(input)
        if not input then return end

        local data = {
            title = input,
            date = os.date("%Y-%m-%d"),
            timestamp = os.date("%Y-%m-%dT%H:%M:%S-06:00")
        }

        local lines = templates.render(chosen_template, data)
        if not lines then
            vim.notify("No template found for key: " .. chosen_template, vim.log.levels.WARN)
            return
        end

        vim.api.nvim_buf_set_lines(0, 0, 0, false, lines)
    end)
end
```

Next, we call the `vim.ui.input` to open a floating text prompt that is prefilled with
the default title. Once you submit it, it creates a table of the title and formatted
timestamps and passes it to our rendering engine, then uses `vim.api.nvim_buf_set_lines`
to inject the lines at the top of the buffer.

#### init.lua - User command setup

```lua
function M.setup(opts)
    vim.api.nvim_create_user_command("Temp", function(args)
        insert_template(args.args)
    end, {
        nargs = "?",
        complete = function()
            local keys = {}
            for k, _ in pairs(templates.registry) do
                table.insert(keys, k)
            end
            return keys
        end
    })
end

return M
```

Finally, we use the `M.setup()` function to register the `:Temp` user command. We then
set `nargs = "?"` so we have optional arguments, and add a complete function that 
autocompletes the template key when you hit Tab. This isn't really necessary, but
I think it's a nice additional touch.

### Conclusion

Building temp.nvim took a minor afternoon annoyance and turned it into a good project
and something that helps my workflow. Now creating a new blog post or dropping 
file documentation for Python is a simple command, which keeps the friction of publishing
code and blog posts very low. 

If you want to see the full plugin's code and instructions for installation, visit [here](https://github.com/adnair2024/temp.nvim).

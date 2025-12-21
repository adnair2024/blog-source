+++
date = '2025-12-21T11:56:44-06:00'
draft = false
title = 'my-markdown'
type = 'post'
tags = ['neovim', 'coding', 'markdown']
categories = ['Neovim']
+++

### My relationship with markdown

I use markdown every single day. It's become an essential part of my workflow,
and this started with note taking apps like Obsidian and Ulysses. When I found
Neovim, everything changed. I started just making todo lists in markdown, within
Neovim, and since all my notes were `.md` files, it was a really easy and natural
way to organize my thoughts and things I had to do without switching context.

I realized pretty quickly that I was running into a problem; creating and
toggling markdown checkboxes was becoming tedious. Every time I wanted to create
one, I had to insert this sequence of unfortunately placed characters `- [ ]` in
all of my notes, and that took time. It was also really easy to mess up when
deleting, so I was getting aggravated when my markdown preview wouldn't render
my checkboxes because I accidentally had two spaces between the brackets instead
of one. I wanted to fix this issue, since a decent part of my time was being 
shifted from note-taking to repairing my various typos.

### The Lua solution

That's why I created `my-markdown`; I wanted an easy way to toggle between the
three states of progress that markdown checkboxes support:

- [ ] *Todo* `- [ ]`
- [-] *In Progress* `- [-]`
- [x] *Completed* `- [x]`

That's really all `my-markdown` does; it lets the user cycle between the states
quickly with a single key bind, so they don't have to manually go and edit the
checkbox every time. The plugin automatically detects the checkbox as long as 
your cursor is on the same line as the checkbox, and it will go and cycle through
the states. Below is the rotation of the states in the order that the plugin
handles them.


| Current State | Visual | Next State | Purpose | 
| --- | --- | --- | --- |
| Todo | `[ ]` | In Progress | For things I need to do, but haven't started | 
| In Progress | `[-]` | Done | For things I am currently working on |
| Done | `[x]` | Todo | For things I finished |


### Technical Deep-dive

I needed my neovim plugin to do three things:

1.  *Read* the line my cursor was currently on
2.  *Identify* the state of the checkbox if any
3.  *Swap* the character and save the line

Thankfully, Lua has great built-in pattern matching, so I used it to make a 
really simple conditional loop. Below is the core logic:

```lua
local line = vim.api.nvim_get_current_line()

-- 1. Check if the line has an empty checkbox [ ]
if line:match("%[%s%]") then
    line = line:gsub("%[%s%]", "[-]", 1) -- Move to In Progress

-- 2. Check if it's already In Progress [-]
elseif line:match("%[%-%]") then
    line = line:gsub("%[%-%]", "[x]", 1) -- Move to Done

-- 3. Check if it's completed [x]
elseif line:match("%[x%]") then
    line = line:gsub("%[x%]", "[ ]", 1)  -- Reset to Todo
end

vim.api.nvim_set_current_line(line)
```

The first version of this plugin actually ended up ruining my notes, so I had to
make some changes. First, I had to escape the brackets since in Lua they're used
to define sets. I used `%[%s%]`, to clearly show that I was looking for a literal
`[`, followed by a space, ended with a `]`.

#### Why I'm using `gsub`

I chose `gsub` (or global substitution) because of its precision. In the lines
where I substitute, I use `1` as the third argument. This is to ensure that 
even if a line has multiple checkboxes (like subtasks), only the first box on 
the line is toggled. This stops the plugin from accidentally toggling all my
notes as done (again).

#### Ease of use

To make the plugin easy to use, I wrapped the logic in a module (`M.toggle_checkbox`)
and exposed it so I could bind it to a key. Now instead of entering Insert mode,
navigating to the brackets, deleting the space, and typing `x`, then escaping
Insert mode back to Normal mode, I could just do `leader key + mt`, and everything
would be done without me ever leaving normal mode. To wrap it up, I abbreviated
`my_markdown` to `mm`; a quick name for a plugin designed to speed up workflows.

### How to Install

The easiest way to get `mm` into your workflow is with lazy.nvim. The code block
below can be copy and pasted to your plugin list.

```lua
{
    'adnair2024/mm',
    config = function()
        local mm = require('mm')
        
        -- Default Toggle: Todo -> In Progress -> Done
        vim.keymap.set('n', '<leader>mt', mm.toggle_checkbox, { desc = "Toggle Markdown Checkbox State" })
        
        -- Optional: Jump straight to Pending/In Progress
        vim.keymap.set('n', '<leader>mp', mm.set_pending, { desc = "Set Checkbox to Pending [-]" })
    end
}
```

The two mappings I have currently are for toggling through the cycles, and one
for immediately setting it to pending, just for ease of use. The `desc` is just
to build a habit, and since I use `which-key.nvim`, it automatically pops
up when you hit your leader key.

### Wrapping up

This plugin was about more than saving four keystrokes; it was also about creating
a tool I could use many times a day rather than forcing my brain to get used to
an arduous sequence of keystrokes. If you spend as much time in markdown files as
I do, I hope `mm` makes your note-taking and organizing simpler.

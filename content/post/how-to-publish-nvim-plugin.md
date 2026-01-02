+++
date = '2025-12-29T10:51:04-06:00'
draft = false
title = 'How to Publish a Neovim Plugin'
type = 'post'
tags = ['neovim', 'coding', 'markdown', 'plugins']
categories = ['Neovim']
+++

## Turning snippets into plugins that everyone can use

  * [Table of Contents](#turning-snippets-into-plugins-that-everyone-can-use)
    * [The Structure of a Neovim Plugin](#the-structure-of-a-neovim-plugin)
    * [Logic](#logic)
    * [Dependencies and what they can do for you](#dependencies-and-what-they-can-do-for-you)
    * [Publishing and Usage](#publishing-and-usage)
    * [Conclusion](#conclusion)

Every Neovim power-user reaches a point when their `init.lua` becomes a mess of
old code snippets. Since I use Markdown files every day for notes, planning and
general writing, my breaking point was functions for Markdown formatting. 
I was tired of manually aligning Markdown tables and manually creating
a Table of Contents for each file, along with streamlining embedding of 
hyperlinks and images. I decided to extract that logic into a
dedicated plugin: [md-helper.nvim](https://github.com/adnair2024/md-helper.nvim). 

If you've ever wanted to publish your own plugin for Neovim, I'll be using my
plugin as a case study for the logic and structure required for smoothly deploying
a plugin to GitHub.

### The Structure of a Neovim Plugin

A plugin is a directory on GitHub that Neovim knows how to look into and search.
The most important part is the `lua/` directory. When you create this folder and 
add a directory for your plugin like `lua/md-helper`, users can access the md-helper 
plugin anywhere in Neovim by calling `require ("md-helper")`. I named my GitHub 
repository `md-helper.nvim` for discovery purposes, while leaving the internal
module name as `md-helper`.

### Logic

When building a Neovim plugin, thinking in terms of the buffer is important. Here
is the logic behind the core of the plugin:

-   The Table of Contents - `(generate_toc)`
    -   Aim: Scan a file and create clickable nested links to the Markdown
        headers found within it.
    -   Logic: The function scans every line of the selected buffer, and counts
        the number of `#` symbols to determine the depth (e.g. `###` is a level
        3 header). 
    -   Link: To make the items clickable, the text "My Header" must become
        `(#my_header)`. My code handles this "slugification" by converting the 
        string to lowercase, stripping special characters, and replacing spaces 
        with hyphens to match standard Markdown parsers.

Below the following code block, I'll supply a screenshot of the output of the
`generate_toc` command.

```lua
M.generate_toc = function()
    local lines = vim.api.nvim_buf_get_lines(0, 0, -1, false)
    local toc = { "## Table of Contents", "" }
    local found_header = false

    for _, line in ipairs(lines) do
        -- Match lines starting with 1 to 6 '#' characters
        local level, title = line:match("^(#+)%s+(.*)$")
        if level then
            found_header = true
            local depth = #level
            -- Ignore the TOC header itself if it already exists
            if title ~= "Table of Contents" then
                -- Strip markdown links from title: [Name](URL) -> Name
                local clean_title = title:gsub("%[([^%]]+)%]%s*%b()", "%1")
                
                -- Convert clean title to a slug (e.g., "My Heading" -> "my-heading")
                local slug = clean_title:lower():gsub("%s+", "-"):gsub("[^%w%-]", "")
                
                -- Indent based on heading level (2 spaces per level)
                local indent = string.rep("  ", depth - 1)
                table.insert(toc, string.format("%s* [%s](#%s)", indent, clean_title, slug))
            end
        end
    end

    if not found_header then
        print("No headers found to create a TOC!")
        return
    end

    -- Insert the TOC at the current cursor position
    vim.api.nvim_put(toc, "l", true, true)
end
```

Output: 
-   ![output-toc](https://i.imgur.com/gVBqDBJ.png) 


-   Dynamic Table Creation
    -   Aim: Eliminate the need to manually align pipes to create a markdown table.
    -   Logic: I used `vim.ui.input` to ask the user for "Rows" and "Columns". The
        plugin then creates a string grid with the specified number of pipes and 
        separators. Then it uses `vim.api.nvim_put` to insert the text block
        containing the table where your cursor is located.

```lua
M.create_table = function()
    vim.ui.input({ prompt = "Number of Rows: " }, function(rows)
        if not rows or rows == "" then return end
        vim.ui.input({ prompt = "Number of Columns: " }, function(cols)
            if not cols or cols == "" then return end
            
            local r, c = tonumber(rows), tonumber(cols)
            if not r or not c then 
                print("Error: Rows and Columns must be numbers")
                return 
            end

            local lines = {}
            -- Build Header: |   |   |
            table.insert(lines, "| " .. string.rep("  | ", c))
            -- Build Separator: |---|---|
            table.insert(lines, "| " .. string.rep("--- | ", c))
            -- Build Rows
            for _ = 1, r do
                table.insert(lines, "| " .. string.rep("  | ", c))
            end

            vim.api.nvim_put(lines, "l", true, true)
        end)
    end)
end
```

-   Image and Link Wrappers
    -   Aim: Reduce time taken to create an image embed or hyperlink.
    -   Logic: These wrappers use Lua's built-in string formatting to take two
        inputs (like description and URL) and drop the text block where your
        cursor is sitting.

```lua
-- FUNCTION: URL Wrapper
M.create_link = function()
    vim.ui.input({ prompt = "Display Text: " }, function(text)
        if not text then return end
        vim.ui.input({ prompt = "URL: " }, function(url)
            if url then
                insert_text(string.format("[%s](%s)", text, url))
            end
        end)
    end)
end

-- FUNCTION: Image Wrapper
M.create_image = function()
    vim.ui.input({ prompt = "Alt Description: " }, function(alt)
        if not alt then return end
        vim.ui.input({ prompt = "Image URL/Path: " }, function(path)
            if path then
                insert_text(string.format("![%s](%s)", alt, path))
            end
        end)
    end)
end
```

You may notice that we use a function called `insert_text`. This wraps the
`vim.api.nvim_put` function because it allows for more precise control over 
cursor placement and for undo-history compared to the usual "append" commands with
respect to the current line state and indentation. This ensures that when a 
link or images are generated, it is added into the buffer with indentation and 
current line state kept in mind. Below, I've added the `insert_text` function so
you can understand how it works

```lua
local function insert_text(text)
    local row, col = unpack(vim.api.nvim_win_get_cursor(0))
    -- row is 1-indexed for the cursor, but 0-indexed for the buffer API
    vim.api.nvim_buf_set_text(0, row - 1, col, row - 1, col, {text})
end
```

### Dependencies and what they can do for you

The best part about the Neovim community is that you'll never have to reinvent
the wheel. I use a plugin called [render_markdown.nvim](https://github.com/MeanderingProgrammer/render-markdown.nvim) to 
display the Markdown elements (links, tables, image links) within Neovim. This way,
once `md_helper.nvim` creates the table, `render_markdown.nvim` makes it visually
appealing. This is why I think it's a great plugin to have as a dependency for 
`md_helper.nvim`. Below I'll add screenshots of markdown tables with and without the
dependency plugin:

![without](https://i.imgur.com/KjvPIwr.png)![with](https://i.imgur.com/VqHsjhk.png)

-   *The first image shows the raw markdown table, and the second one shows the 
    markdown table rendered with `render_markdown.nvim`.*

### Publishing and Usage

Once I tested it locally, I proceeded to push my code to GitHub. To make my project
more professional, I added a detailed README.md that included a configuration
block for `Lazy.nvim`.

```lua
{
    "adnair2024/md-helper.nvim",
    dependencies = { 
        "MeanderingProgrammer/render-markdown.nvim",
    },
    ft = { "markdown" }, -- Load only on markdown files
    config = function()
        local md_helper = require("md-helper")

        -- Recommended Keybindings
        -- You can customize these keys to whatever fits your workflow
        
        -- Generate Table of Contents
        vim.keymap.set("n", "<leader>mc", md_helper.generate_toc, { desc = "Markdown: Create TOC" })
        
        -- Create Table (Prompts for Rows x Cols)
        vim.keymap.set("n", "<leader>mt", md_helper.create_table, { desc = "Markdown: Table" })
        
        -- Insert Link (Prompts for Text & URL)
        vim.keymap.set("n", "<leader>ml", md_helper.create_link, { desc = "Markdown: Link" })
        
        -- Insert Image (Prompts for Alt Text & Path)
        vim.keymap.set("n", "<leader>mi", md_helper.create_image, { desc = "Markdown: Image" })
    end
}
```

I also made sure to add instructions in the README.md on how to set up whichkey
keybinds, in case someone uses them, as I do. Now that it's on GitHub and Lazy.nvim
automatically pulls from GitHub, we are ready to go. Anyone who uses a Lazy.nvim
`init.lua` can use my plugin directly in Neovim, by calling `adnair2024/md-helper.nvim`.
It's also important to note that the folder structure is strict, and should be
the following, since Neovim always looks into the `lua/` folder located into the
root directory of your repository.

```
|- md-helper.nvim/
    |-   lua/
        |- md-helper/
            |-   init.lua - All functions stored here
    |-   README.md
```

### Conclusion

Publishing `md-helper.nvim` taught me how simple and lightweight it is to develop
plugins for Neovim. You just need a tool that solves a specific problem for you
and by publishing it as a plugin to GitHub, you can help others who have this
problem as well. By following the `lua/` directory standard and also adding
in existing dependencies, you can build something simple that feels like a native
Neovim feature and not an external plugin.

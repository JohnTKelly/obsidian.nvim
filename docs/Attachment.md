## Save location

Option for attachment location is `opts.attachments.folder`

1. for vault root, set it to `/`.
2. for fixed folder, set it to `/folder-name`.
3. for same folder as current file, set it to `./`
4. for sub folder in current folder, set it to `./folder-name`

## Add attachment

- If `source` is a local path (or `file://` URI), the file is copied.
- If `source` is an `http(s)` URL, the file is downloaded with `curl`.
- The destination path is always resolved by `api.resolve_attachment_path()`.

When called without an argument, obsidian.nvim uses `opts.attachments.pick` (if set).

`attachments.pick` receives a callback that you can call with a filepath/URL once your picker resolves a choice. It can also return a filepath/URL directly.

```lua
require("obsidian").setup {
  attachments = {
    pick = function(add)
      local src = vim.fn.input "Attachment path or URL: "
      if src and src ~= "" then
        add(src)
      end
    end,
  },
}
```

### Picker examples

Pick with `snacks.explorer`:

```lua
attachments = {
  pick = function(add)
    local ok, Snacks = pcall(require, "snacks")
    if not ok then
      return
    end

    Snacks.picker.explorer {
      title = "Pick attachment",
      focus = "list",
      confirm = function(picker, item)
        picker:close()
        if item and item.file then
          add(item.file)
        end
      end,
    }
  end,
}
```

Pick with a terminal file manager:

```lua
attachments = {
  pick = function(add)
    local tmp = vim.fn.tempname()
    vim.system({ "bash", "-c", "ranger --choosefile='" .. tmp .. "'" }):wait()
    if vim.uv.fs_stat(tmp) then
      local lines = vim.fn.readfile(tmp)
      if lines[1] then
        add(lines[1])
      end
    end
  end,
}
```

Pick from URLs only:

```lua
attachments = {
  pick = function(add)
    local url = vim.fn.input "URL: "
    if url and url ~= "" then
      add(url)
    end
  end,
}
```

## Open

Attachment opening is by default controlled by `:h vim.ui.open`, customize it like following:

```lua
vim.ui.open = (function(overridden)
  return function(uri, opt)
    if vim.endswith(uri, ".png") then
      vim.cmd("edit " .. uri) -- early return to just open in neovim
      return
    elseif vim.endswith(uri, ".pdf") then
      opt = { cmd = { "zathura" } } -- override open app
    end
    return overridden(uri, opt)
  end
end)(vim.ui.open)
```

Put any where in you config that loads before you open attachments, a good place could be `opts.callback.enter_note`

goyo.vim ([고요](http://en.wiktionary.org/wiki/고요하다))
=========================================================

Distraction-free writing in Vim.

Installation
------------

Use your favorite plugin manager.

- [vim-plug](https://github.com/junegunn/vim-plug)
  1. Add `Plug 'junegunn/goyo.vim'` to .vimrc
  2. Run `:PlugInstall`

Usage
-----

- `:Goyo`
    - Toggle Goyo
- `:Goyo [dimension]`
    - Turn on or resize Goyo
- `:Goyo!`
    - Turn Goyo off

The window can be resized with the usual `[count]<CTRL-W>` + `>`, `<`, `+`,
`-` keys, and `<CTRL-W>` + `=` will resize it back to the initial size.

### Dimension expression

The expected format of a dimension expression is
`[WIDTH][XOFFSET][x[HEIGHT][YOFFSET]]`. `XOFFSET` and `YOFFSET` should be
prefixed by `+` or `-`. Each component can be given in percentage.

```vim
" Width
Goyo 120

" Height
Goyo x30

" Both
Goyo 120x30

" In percentage
Goyo 120x50%

" With offsets
Goyo 50%+25%x50%-25%
```

Configuration
-------------

- `g:goyo_width` (default: 80)
- `g:goyo_height` (default: 85%)
- `g:goyo_linenr` (default: 0)

### Callbacks

By default, [vim-airline](https://github.com/bling/vim-airline),
[vim-powerline](https://github.com/Lokaltog/vim-powerline),
[powerline](https://github.com/Lokaltog/powerline),
[lightline.vim](https://github.com/itchyny/lightline.vim),
[vim-signify](https://github.com/mhinz/vim-signify),
and [vim-gitgutter](https://github.com/airblade/vim-gitgutter) are temporarily
disabled while in Goyo mode.

If you have other plugins that you want to disable/enable, or if you want to
change the default settings of Goyo window, you can set up custom routines
to be triggered on `GoyoEnter` and `GoyoLeave` events.

```vim
function! s:goyo_enter()
  if executable('tmux') && strlen($TMUX)
    silent !tmux set status off
    silent !tmux list-panes -F '\#F' | grep -q Z || tmux resize-pane -Z
  endif
  set noshowmode
  set noshowcmd
  set scrolloff=999
  Limelight
  " ...
endfunction

function! s:goyo_leave()
  if executable('tmux') && strlen($TMUX)
    silent !tmux set status on
    silent !tmux list-panes -F '\#F' | grep -q Z && tmux resize-pane -Z
  endif
  set showmode
  set showcmd
  set scrolloff=5
  Limelight!
  " ...
endfunction

autocmd! User GoyoEnter nested call <SID>goyo_enter()
autocmd! User GoyoLeave nested call <SID>goyo_leave()
```

More examples can be found here:
[Customization](https://github.com/junegunn/goyo.vim/wiki/Customization)

FAQ
---

> _"My custom colors are lost when I exit Goyo!"_

That's because Goyo restores the base color scheme using `:colorscheme
CURRENT_COLOR_SCHEME` command on exit, and it resets your tweaks. Goyo can try
to remember all your customizations and restore them on exit, but it doesn't,
because there is a better, more robust way to address the issue.

The real problem here is that you will lose all your changes when you switch
between color schemes, even when you're not using Goyo.

```vim
" In your Vim configuration file
" - Base color scheme
colorscheme molokai

" - Your color customizations
hi LineNr ctermfg=red guifg=red
```

It works, only when you stick to a single color scheme. When you switch
between color schemes,

```vim
" Switch to another color scheme
colorscheme Tomorrow-Night

" Switch back to the original one
colorscheme molokai
```

And all the customizations you have made are lost.

What you should to do is to customize the colors on `autocmd ColorScheme`,
which is automatically triggered whenever you change color schemes.

```vim
function! s:tweak_molokai_colors()
  " Your molokai customizations
  hi LineNr ...
  hi FoldColumn ...
endfunction

autocmd! ColorScheme molokai call s:tweak_molokai_colors()

colorscheme molokai
```

Inspiration
-----------

- [LiteDFM](https://github.com/bilalq/lite-dfm)
- [VimRoom](http://projects.mikewest.org/vimroom/)

Pros.
-----

1. Works well with splits. Doesn't mess up with the current window arrangement
1. Works well with popular statusline plugins
1. Prevents accessing the empty windows around the central buffer
1. Can be closed with any of `:q[uit]`, `:clo[se]`, `:tabc[lose]`, or `:Goyo`
1. Can dynamically change the width of the window
1. Adjusts its colors when color scheme is changed
1. Realigns the window when the terminal (or window) is resized or when the size
   of the font is changed
1. Correctly hides colorcolumns and Emojis in statusline
1. Highly customizable with callbacks

License
-------

MIT


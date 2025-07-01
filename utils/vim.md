---
layout: default
title: Vim notes
categories: software
---

#### Symbols and meanings
- `%` &rarr; current file. An example:  `:so %` &rarr; Source the current file
- `$` &rarr; end of line
-  `.` &rarr; Current line An example: `:.!sh` &rarr; Pipe current line to `sh` and replace it with the output

Entering `!!` in normal mode is translated to  `:.!` I. e. Typing `!!date` in normal mode replaces current line with the date.

#### Tips

- `:e[dit]` &rarr;	Edit the current file. This is useful to re-edit the current file, when it has been changed outside of Vim. `:e!` Force reload file
- `:help index` &rarr; Get all default mappings

#### Navigation

- `h`  `j`  `k`  `l` &rarr; left, down, up, right
- `*` &rarr; Next whole word under cursor (previous `#`)
- `e` &rarr; Forward to the end of word. `E` can contain punctuation
- `w` &rarr; Move forward to the beginning of a word. `W` Same as `w`, but special characters are treated as part of a word.
-  `b` &rarr; Works as `w`, but backwards 
-  `{`,`}` &rarr; Jump by paragraphs
- `(`,`)` &rarr; Jump by sentences
- `G` &rarr; Jump to the end of the file
- `1G` &rarr; Jump to the beginning of the file (same as `gg`)
- `50G` &rarr; Jump to line 50
- `0` &rarr; Beginning of line
- `_` or `^` &rarr; first non-blank character of the line
- `g_`  &rarr; last non-blank character of the line
- `fX` &rarr; next character `X`. `FX` previous. `;`   repeat , `,`   repeat in reverse
-  `tX` &rarr; tili next `X` (similar to above, but the cursor is before `X`)
- `H` &rarr; Jump to the top of the screen
- `M` &rarr; Jump to the middle of the screen
- `L` &rarr; Jump to the bottom of the screen

#### Scrolling

* `10 <PageUp>` or `10<CTRL-B>` &rarr; Move 10 pages up
* `5 <PageDown>` or `5<CTRL-F>` &rarr; Move 5 pages down.
- `zz`  &rarr; scroll the line with the cursor to the center of the screen
- `zt`  &rarr; to the top
- `zb`  &rarr; to the bottom

### Terminal buffers

- `:te[rm[inal]] command`
- `:b#` switch buffer
- `:ls` list buffers
- `:buff 1` or `:b1` switch to buffer 1

### List of the commands

Common meaning of letters in the commands
- `w` &rarr; word
- `i` &rarr; inner

| Command |  |
| ---: | :--- |
| `dd` |  Delete one line |
| `d` | Delete selection |
| `x` | Delete character under cursor |
| `d+` | Delete 2 lines |
| `:%d` or :`1,$d` |  Delete the whole of the file |
| `dw`, `diw` | Delete what that the cursor is over |
| `di(` | Delete inner brackets. `da(` &rarr; including brackets |
| `:r[ead] !date` | Execute commend and put content into editor |
| `.` | Repeat the last operation |
| `gU` | Uppercase the selection, `gu`  &rarr; lower |
| `%` | Jump to matching bracket `{ }` `[ ]` `( )` |
| `:%!column -t` | Put text in columns |
| `:%!sort` | Sort the whole file |
| `:'<,'>!grep text` | Keep lines that contains `text` |
| `:'<,'>!sort` | Sort selected lines |
| `:eariler 1m` | State from the 1 min before |
| `ga` | Display hex, ascii value of character under cursor |
| `g8 ` | Display hex value of utf-8 character under cursor |
| `ciw` | Change inner word |
| `yiw`  |  Yank inner word |
| `viwp`  | Select word and then replace it with previously yanked text |
| `rX` |	replace every character in selection or under cursor with `X` |
| `guiw` | Lower case word |
| `guu` | Lowercase line |
| `gUU` | Uppercase line |
| `=` | Indent the selection |
| `=%` | Indent the current braces |
| `G=gg` |  indent entire document |
| `ZZ` | Write current file, if modified, and exit (same as `:wq`) |
| `ZQ` | Quit current file and exit (same as `:q!`) |
| `:so %` | Source current file |
| `@:` | Execute last command again |

#### Status line

```vim
:set statusline=%<%f%h%m%r%=%b\ 0x%B\ \ %l,%c%V\ %P
```

#### Visual Mode

- `V` &rarr; Selection by lines
- `v` &rarr; Selection follows the cursor
- `Ctrl+v` &rarr;  Block selection
   - When selected block you can applay changes to each line by typing `I` editing and finally pressing `Esc`

#### Enter Insert Mode

- `I` &rarr; At the first non white character of the line
- `i` &rarr; On the left of the cursor
- `A` &rarr; At the very end of the line
- `a` &rarr; On the right of the cursor
- `c` &rarr;  Delete selection and enter insert mode
- `o` &rarr; Create new line below and enter insert mode
- `O` &rarr; Create new line above and enter insert mode

#### Split the editor

- `:sp <filename>` &rarr; Vertically
- `:vs <filename>` &rarr; Horizontally
- `:set splitbelow`, `:set splitright`

#### Markers

-  `:marks` &rarr;  list of marks 
- `ma` &rarr; set current position for mark `a`
- `` `a`` &rarr; jump to the cursor position of mark `a`
- `'a` &rarr;  jump to the beginning of a line of a mark `a`
- ``y`a`` &rarr; yank text to position of mark `a` 
*  ``` `` ``` &rarr; Return to the cursor position before the latest jump
* `` `.``  &rarr; Jump to the last changed line.

### Recording macros

1. `qa` &rarr; Start recording macro under letter `a`
2. `q` &rarr; Stop recording 
3. `@a` &rarr; Play the macro saved under letter a
4. `@@` &rarr; Play the last macro

#### Searching

- `:%s/Plug.*$//` &rarr; Search and delete all lines that starts from Plug
- `:%s/foo/bar/gc` &rarr; Replace all occurrence of foo by bar with confirmation
- `'<,'>:s/find/replacewith/` Replace selection
- `/pattern` &rarr; search for pattern then enter and `n` next `N` previous match
- `?pattern` &rarr; search backward for pattern

### Registers

- `:reg` &rarr; print all registers
- `"ap` &rarr; paste the register `a` , if the macro is recorded then it will paste it
- `"xy` &rarr; yank into register `x` 
- `:let @a = "kkll"` &rarr; set a macro from the command mode
- `:let @A='i'`  &rarr;  append to register `a`
- `:%normal @a`  &rarr; execute the macro on all lines of the current file
- `:'<,'>normal @a`  &rarr; execute the macro on a visually selected lines
- `:10,20 normal @a` &rarr; execute the macro for lines from 10 to 20
- `:g/pattern/ normal @a` &rarr; Search for pattern and execute macro for it

#### Functions - An example

Function definition

```vim
function! CalculateAge()
    normal 03wdei^R=2012-^R"^M^[0j 
endfunction
```

Key banding to function

```vim
nnoremap <leader>a :call CalculateAge()<CR>
```

Preloading vim with macros like

```vim
let @a='03wdei^R=2012-^R"^M^[0j'
```

Call function from the command mode

```
:call CalculateAge()
```

#### Configuration

The config file is located at `.config/nvim/init.vim`

```vim
if empty(glob('~/.local/share/nvim/site/autoload/plug.vim'))
  silent !curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs
    \ https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
  autocmd VimEnter * PlugInstall --sync | source $MYVIMRC
endif

call plug#begin('~/.vim/plugged')

  Plug 'junegunn/fzf'

  Plug 'neovim/nvim-lspconfig'
  Plug 'neoclide/coc.nvim', {'branch': 'release'}
  
  Plug 'vim-airline/vim-airline'
  Plug 'vim-airline/vim-airline-themes'
  
  Plug 'vim-syntastic/syntastic'
  Plug 'tokorom/syntastic-swiftlint.vim'
  
call plug#end()
```

- `set relativenumber`
- `set encoding=utf-8`
- `syntax on` 
- `map za :FZF<CR>` &rarr; fuzzy finder over `za`

Indentation setup 

- `set tabstop=2 shiftwidth=2 expandtab`
- `filetype plugin indent on`

```vim
let g:syntastic_swift_checkers = ['swiftlint', 'swiftpm']
lua << EOF
  local lspconfig = require('lspconfig')
  lspconfig.sourcekit.setup{}
EOF
```

#### Rename current file


```vim

function! RenameFile()
    let old_name = expand('%')
    let new_name = input('New file name: ', expand('%'), 'file')
    if new_name != '' && new_name != old_name
        exec ':saveas ' . new_name
        exec ':silent !rm ' . old_name
        exec ':bd ' . old_file
        redraw!
    endif
endfunction
map <leader>n :call RenameFile()<cr>
```



<!--
### Navigation in Vimr
- jump to the next empty line (the next paragraph)`}` 
* Recording macros
    * `"xp` pastes the contents of the register `x`.


* `~ `      : invert case (upper->lower; lower->upper) of current character
* `gf `     : open file name under cursor (SUPER)

* `ggg?G`  : rot13 whole file
* `xp`      : swap next two characters around
* `CTRL-A,CTRL-X` : increment, decrement next number on same line as the cursor
* `CTRL-R=5*5`    : insert 25 into text



* `'.`       : jump to last modification line (SUPER)
* *`.*       : jump to exact spot in last modification line
* `<C-O>`    : retrace your movements in file (backward)
* `<C-I>`    : retrace your movements in file (forward)
* `:ju(mps)` : list of your movements {{help|jump-motions}}
:history : list of all your commands


 Sorting with external sort
:%!sort -u           : contents of the current file is sorted and only unique lines are kept
:'v,'w!sort          : sort from line marked v thru lines marked w
:g/^$/;,/^$/-1!sort  : sort each block (note the crucial ;)

!1} sort             : sorts paragraph; this is issued from normal mode!)

:wn           : write file and move to next (SUPER)
:bd           : remove file from buffer list (SUPER)
:sav php.html : Save current file as php.html and "move" to php.html
:w /some/path/%:r   : save file in another directory, but with the same name
:e #          : edit alternative file
:args         : display argument list
:n            : next file in argument list
:prev         : previous file in argument list
:rew          : rewind to first file in argument list
:ls           : display buffer list
:bn           : next buffer
:bp           : previous buffer
:brew         : rewind to first buffer in buffer list
:tabe         : open new tab page (Ctrl-PgUp, Ctrl-PgDown for next/previous tab)
:tabm n       : move tab to position n (0=leftmost position)

# editing a register/recording
"ap
<you can now see register contents, edit as required>
"add
@a




Ctrl-D  move half-page down
Ctrl-U  move half-page up
Ctrl-B  page up
Ctrl-F  page down
Ctrl-O  jump to last (older) cursor position
Ctrl-I  jump to next cursor position (after Ctrl-O)
Ctrl-Y  move view pane up
Ctrl-E  move view pane down

n   next matching search pattern
N   previous matching search pattern


g*  next matching search (not whole word) pattern under cursor
g#  previous matching search (not whole word) pattern under cursor

de — Delete to the end of the word
^R= — Insert the contents of the special = register, which accepts an expression to evaluate

gUgn - uppercase
gn
n move to the next match


Undo and redo

You can use u to undo the last change. CTRL-R redoes a change that has been undone. U returns the current line to its original state.
You can use g- or g+ to go between text-states. 
Search and replace
* `\vpattern` - 'very magic' pattern: non-alphanumeric characters are interpreted as special regex symbols (no escaping needed)
* `n` - repeat search in same direction
* `N` - repeat search in opposite direction

* `:noh` - remove highlighting of search matches
Search in multiple files
* `:vimgrep /pattern/ {file}` - search for pattern in multiple files

* e.g. `:vimgrep /foo/ **/*`
* `:cn` - jump to the next match
* `:cp` - jump to the previous match
* `:copen` - open a window containing the list of matches


folding from selection
`: '<,'>fo`

`:help folding`

set foldmethod=syntax
`set foldlevel=1`
`set foldclose=all`


Sequence forfolding lines `Shift+V:fo`
`:set foldmethod=syntax` intent, 
`zo` unfolding
`za` toogle folding
`zf#j` creates a fold from the cursor down # lines.

:CheckHealth
-->

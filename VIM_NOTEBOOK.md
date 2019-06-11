# A VIM Notebook
Theory, tips, and tricks after reading Drew Neil's Practical VIM

Tips and tricks
An unordered list of tips picked up along the way

---
#### Beginners: if you need to force yourself to stop using the arrow keys:
```
nnoremap <Left> :echo "Use h"<CR>
nnoremap <Right> :echo "Use l"<CR>
nnoremap <Up> :echo "Use k"<CR>
nnoremap <Down> :echo "Use j"<CR>
```

---

#### Global yanking with the OS clipboard:
```
set clipboard=unnamed on .vimrc for
```

---

#### Working with registers: 
1. If you keep yanking code and messing it up somehow with `d` or `x`, you can always use the yank dedicated register `"0p` will tell vim to "put from yank register and not from the unnamed one"
2. You can also cut to a specific register: `"adw` will `delete` a `word` to the "a" register.
3. Last thing - you can interact in the same way with the OS clipboard! using 
`"+{command}` will tell vim to do the command to/from the clipboard register.

---

#### Commenting out code:
Can be done with:
1. get into vertical-block mode by using `ctrl-V`. 2. mark the lines you want to modify. 3. `shift-I` `#` `space` `esc`
the best way to add a prefix to lines. adding a suffix also works if after `ctrl-V` you use `$` and replace `shift-I` with `shift-A`

2. using the super awesome [Vim-Commentary](https://github.com/tpope/vim-commentary) plugin by tpope.

---

#### Searching
Vim uses `*` to search the current word on normal mode, but when using visual mode you'd expect it would search your highlighted text, but it doesn't, fix it!
```
" Operating on a complete search match
xnoremap * :<C-u>call <SID>VSetSearch()<CR>/<C-R>=@/<CR><CR>
xnoremap # :<C-u>call <SID>VSetSearch()<CR>?<C-R>=@/<CR><CR>
Some search tips:
1. You can specify search direction in VIM; use `/` to search forward and `?` to search backwards (`n` and `N` will iterate over results backwards too)
2. Highlight search results with setting `set hlsearch`
3. After using the highlight, it can be a annoying if it stays, disable using `:noh` or take it another step and map to something like `Ctrl+l`: `nnoremap <C-l> :noh<CR>`
4. Find number of search matches (without a plugin): `:%s///gn` (telling vim to search and replace without contents makes it use the last search and then tells it to count.
5. Want to position cursor on the *end* of the search result instead of the beginning? use `e`: 
`/searchphrase/e`. Forgot and already searched? use `//e` - becomes useful when you'd like to operate on phrases that can be part of words.
6. Ignore / force search case sensitivity: use `/c` to ignore case, and `/C` to force it. This overrides the setting of `ignorecase`
function s:VSetSearch()
 let temp = @s
 norm! gv"sy
 let @/ = '\V' . substitute(escape(@s, '/\'), '\n', '\\n', 'g')
 let @s = temp
endfunction
```
Now, on visual mode, mark a text and press `*` to search forward, and `#` to do the same backwards.

---

#### Search your VIM command history!
`:` then `<Ctrl-f>` you get a history list you can browse
If you want to vim-search it just use `/` and search

---

#### Find case insensitive results
`set ignorecase` if you want vim to

---

#### Better VIM scroll 
(inspired by DAS talks about moving in large chunks, I added the scroll feel):
```
nnoremap <C-j> 10jzz
nnoremap <C-k> 10kzz
```
Maps moving up or down with `Ctrl` pressed, to a bigger motion (`10`) while keeping the line in the center of the screen (`zz`). This gives a better "scroll" feel.

---

#### Mappings
Mapping shell commands is useful, e.g for running tests, this is something usually set right in the session and not as a permanent setting. Something like this:
`nmap tt :! crystal spec <enter>`
Maps `tt` in normal mode to run `crystal spec` and press enter.

---

#### Using the black hole register:
If you yank a text and right before pasting it you use `d` or `c` to remove the text you want replaced you need to go back and yank it again.
One solution is to mark `v` the unwanted text and then `p` on it.
BUT - using the black hole register is a good option too - 
If you want to remove text whether using `x`/`s`/`c`/`d` use `"_` before
`"` is sending your yanked / deleted text into a register, by providing `"_` it goes to `/dev/null`, allowing you to keep working with the unnamed register without changing it in the process.

---

#### Using markers and last location
Using `m` is a cool feature if you want to mark a specific location in a file and go back, e.g: `ma` and then after moving around using `'m` will throw you back.
But if you didn't mark anything and just want to jump back to where you've been last, use `''`

---

#### Incrementing
Incrementing numbers in vim: `Ctrl-a`
Decrementing numbers in vim: `Ctrl-x`
Very useful in macros where you want to programmatically run through lists and increment numbers.

---

#### Applying macros on multiple files (programmatically with NERDTree)
Using [NERDTree](https://github.com/scrooloose/nerdtree) is a convenient way to browse through files if you're not familiar with the project (if you are, use https://github.com/Yggdroot/LeaderF)
If you have a macro running on a file, and you'd like to run it on many of them, you can integrate commands jumping to nerdtree and opening the next file:
`<macro-commands>^Whj^M^Wl`
Where `^W` being ctrl-W to switch to window selector, `h` moving left to the file browser, `j` moving down to the next file on the list, `^M` is Enter which opens the file, and again `^Wl` is ctrl-W and `l` moving to the right window again.
Running this combination in a loop can apply the same `<macro-commands>` on a list of files with relative ease.

---

#### Going directly to your last opened file and last location of the cursor!
Open Vim and hit
```
Ctrl+o+o
```

---

#### Editing a read only file and forgot to use `sudo`?
You don't need to leave VIM!
```
:w !sudo tee %
```
Of course this can be aliased to anything e.g `sudowrite`

---

#### Delete a line without storing it in the unnamed register
Added this to my .vimrc: `nnoremap DD ""dd` . I find it useful when many times I want to delete before pasting.

---

#### See the file name you're working on:
```
:f
```

---

#### Sorting
Mark your lines and `:sort`
But you can also:
Sort and remove duplications with unique flag:
`:sort u`
Sort in reverse:
`:sort!`
Numeric sort:
`:sort n`
More at http://vim.wikia.com/wiki/Sort_lines

---

#### See your tabs and spaces
```
:set list
```
Useful for yaml and indentation based structures debugging
`:set nolist` to disable
`listchars` has different ways of highlighting tabs spaces etc.
Try `set listchars=tab:..,trail:_,extends:>,precedes:<,nbsp:~` and then insert a tab (when `:set list` is on) and see the changes

---

#### Delete trailing white space on save
```
func! DeleteTrailingWS()
 exe "normal mz"
 %s/\s\+$//ge
 exe "normal `z"
endfunc
au BufWrite * silent call DeleteTrailingWS()
```

#### Reloading .vimrc while working:
```
:so %
```

---

#### Solving merge conflicts with vim fugitive(!)
When in a merge conflict try `:Gdiff` (or `:Gvdiff` to open vertically)
You'll get a 3-way split panes, the middle one is the working file and on the sides the two options: HEAD or target branch, which can be referred to by buffer names `//2` and `//3` (`:ls` will show all buffer full names).
You now have two options:
1. When standing in the working copy you can `diffget <buffer name/number>`
2. Go to other pane and `diffput <buffer name/number>`
I mapped the process to ease the flow like that:
```
" Open a vertical diff
nnoremap <leader>gd :Gvdiff<CR>
" Get the left pane changes
nnoremap gdh :diffget //2<CR>
" Get the right pane changes
nnoremap gdl :diffget //3<CR>
" Stage the changes (will automatically close all buffers leaving you in the center)
nnoremap <leader>gw :Gwrite<CR>
```

---

#### NERDTree
`go` while browsing will open a file without changing the context
`gi` will do the same in a split
`?` will transform the tree pane to the help will all options

---

#### Vim Terminal (Vim8 / neovim)
Open a terminal with :terminal
While using the Terminal go to normal mode with CTRL-W N or 
CTRL-\ CTRL-N
Kill the session with CTRL-W CTRL-C

---



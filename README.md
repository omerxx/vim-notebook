# A VIM Notebook
Theory, tips, and tricks after reading Drew Neil's Practical VIM & An unordered list of tips picked up along the way

---
#### Beginners: if you need to force yourself to stop using the arrow keys:
```vim
nnoremap <Left> :echo "Use h"<CR>
nnoremap <Right> :echo "Use l"<CR>
nnoremap <Up> :echo "Use k"<CR>
nnoremap <Down> :echo "Use j"<CR>
```

---

#### Global yanking with the OS clipboard:
```vim
set clipboard=unnamed
```
Added to .vimrc will make Vim yank to the clipboard, to be pasted anywhere.

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
```vim
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
```vim
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
```vim
Ctrl+o+o
```

---

#### Editing a read only file and forgot to use `sudo`?
You don't need to leave VIM!
```vim
:w !sudo tee %
```
Of course this can be aliased to anything e.g `sudowrite`

---

#### Delete a line without storing it in the unnamed register
Added this to my .vimrc: `nnoremap DD ""dd` . I find it useful when many times I want to delete before pasting.

---

#### See the file name you're working on:
```vim
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
```vim
:set list
```
Useful for yaml and indentation based structures debugging
`:set nolist` to disable
`listchars` has different ways of highlighting tabs spaces etc.
Try `set listchars=tab:..,trail:_,extends:>,precedes:<,nbsp:~` and then insert a tab (when `:set list` is on) and see the changes

---

#### Delete trailing white space on save
```vim
func! DeleteTrailingWS()
 exe "normal mz"
 %s/\s\+$//ge
 exe "normal `z"
endfunc
au BufWrite * silent call DeleteTrailingWS()
```

#### Reloading .vimrc while working:
```vim
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
```vim
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

#### Solving merge conflicts with vim fugitive(!):
When in a merge conflict try `:Gdiff` (or `:Gvdiff` to open vertically)
You’ll get a 3-way split panes, the middle one is the working file and on the sides the two options: HEAD or target branch, which can be referred to by buffer names `//2` and `//3` (`:ls` will show all buffer full names).

You now have two options:
1. When standing in the working copy you can `diffget <buffer name/number>`
2. Go to other pane and `diffput <buffer name/number>`

I mapped the process to ease the flow like that:
```vim
" Open a vertical diff
nnoremap <leader>gd :Gvdiff<CR>
" Get the left pane changes
nnoremap gdh :diffget //2<CR>
" Get the right pane changes
nnoremap gdl :diffget //3<CR>
" Stage the changes (will automatically close all buffers leaving you in the center)
nnoremap <leader>gw :Gwrite<CR>
```

Another useful shortcut is to jump to the next hunk of changes with `[c` or `]c`
Really nice and visible way to solve conflicts!

[I've turned this into a blog post](https://medium.com/prodopsio/solving-git-merge-conflicts-with-vim-c8a8617e3633)

---

#### Zooming into split panes
Natively you can enlarge a pane by `<C-w> _` for horizontal and `<C-w> |` for vertical ones.
I added a script that uses these like Tmux’s zoom-in feature:
```vim
" Pane Zoom
func! PaneZoom()
  res 100
  vert res 1000
endfunc
nnoremap <C-w>z :call PaneZoom()<CR>
```
Now I can `<C-w> z` anywhere. (Revert the split to be even with `<C-w> =`)

---

#### The meaning of life:
```vim
:help 42
```

---

#### Utilize search and replace
Putting aside the `:%s/<text>/<replace text>/g` option and its interactive version `:%s/<text>/<replace text>/gc`, you can use the `gn` tool for quick find and replace:
1. Search for your pattern e.g `/const`
2. Use `cgn` which basically edits the next match `cg` and them jumps to the next one `n`
3. Then you can simply use `.` to repeat the last command on as many matches you want.

http://vimcasts.org/episodes/operating-on-search-matches-using-gn/
https://medium.com/@schtoeffel/you-don-t-need-more-than-one-cursor-in-vim-2c44117d51db

---

#### Movement and surrounding
`w` jumps one word, if you have `svc.cluster.local` these are 3 words. But if you use `W` vim will address word as the characters between 2 spaces, so that `svc.cluster.local` becomes one. Why is it useful? First - better movement around VIM, another example is trying to wrap with parentheses for example, where I can use `ysiW` (using *tpope/vim-surround*) to include exactly what I want

---

#### Save all buffers (and panes):
```vim
:wa
```

---

#### Search code in a project!
Using `fzf` you can `:Ag` to search for any string which immediately shows up results.
BUT better yet, you can send the word under the cursor to it and populate results to a preview pane

---

#### Utilizing Ag the silver searcher
https://github.com/ggreer/the_silver_searcher

This is the mapping that sends the result to `ag` with preview:
```vim
nnoremap <leader>F
  \ :call fzf#vim#ag('', fzf#vim#with_preview({'options': ['--query', expand('<cword>')]}))<cr>
```

---

#### Tabs with sessions

1. Open any number of tabs you wish to work with
2. Type `:mksession` header-files-work.vim and hit enter
3. Your current session of open tabs will be stored in a file header-files-work.vim
4. Close all tabs and Vim
5. Either start vim with your session using : `vim -S header-files-work.vim` or open vim with any other file and enter command mode to type: `:source header-files-work.vim` and *BOOM!*

---

#### Save As

If you work on a file and would like to save it with a new name (like *save as*) you can:
1. `:w <NAME>` would write an external file and let you keep working on the original one
2. `:sav <NAME>` would write an external file and change your current buffer to the new one as well (a more likely scenario)

---

#### Folding

Sometimes it’s visibly comfortable to fold pieces of code.
`z` is the controller here, when combined with `f` it folds the motion added to it (`zf'm` will fold until the marker “m”).
Most intuitive way is to have a visual block marked and just `zf` to fold.
`za` toggles the created fold on and off.
`zd` deletes a fold.

These can be applied automatically with other modes of folding: `:setlocal foldmethod=indent` for example will create basic folding under indentation levels of code.
More helpful docs: https://vim.fandom.com/wiki/Folding

```vim
zf#j creates a fold from the cursor down # lines.
zf/ string creates a fold from the cursor to string .
zj moves the cursor to the next fold.
zk moves the cursor to the previous fold.
za toggle a fold at the cursor.
zo opens a fold at the cursor.
zO opens all folds at the cursor.
zc closes a fold under cursor.
zm increases the foldlevel by one.
zM closes all open folds.
zr decreases the foldlevel by one.
zR decreases the foldlevel to zero -- all folds will be open.
zd deletes the fold at the cursor.
zE deletes all folds.
[z move to start of open fold.
]z move to end of open fold.
```

---

#### Vim marks

An awesome way to jump between sections when editing a file (https://vim.fandom.com/wiki/Using_marks):
1. `ma` marks a line as `a` (or any other letter)
2. Jump to the mark with `'a`
3. TIP: `'` is always a marker of your previous location, so by `''` you can always jump back (kinda like `cd -` in bash)

I looked for a way to see these markers visible (when you have more than 2-3 its hard to remember them), so [there’s a nice plugin](https://github.com/kshenoy/vim-signature)

---

#### Join two lines

If you have a line broken to many parts or just want to join two lines into 1 try Shift + `j`

---

#### Small surrounding Tip

In [vim surround](https://github.com/tpope/vim-surround) when you wrap with `{` or `[` you always get an unwanted space (e.g `{ $var }`) to lose the spaces, use the closing brace (`}`)

---

#### Spanning Vim splits

You can send a vim split to span full screen with `J`:
If you have two splits side by side, and you open another one it stays on the focused side when creating it. To make it span both original splits on the bottom: `<CR-W>J`

---

#### Syntax refresh when Vim messes it up

Refresh syntax highlight when it gets messed up in long files : `:syntax sync fromstart`

---

#### Open your current file (e.g. in the browser)

```vim
:!open % (macOS)
:!xdg-open % (Linux)
```
 so if you’re editing an html file for example it goes straight to the browser

---

#### Lowercasing

If you visually mark a text and press `u` it turns the text to lowercase completely.

---

#### Remove last search result highlighting
Probably not that exciting, but stopping a highlight of a current search, I used to search for a non existing string until today like `/sss` which is dumb..
I just added a map of `nnoremap ss :noh<CR>` which for me `ss` is something I was used to do and also an abbreviation of “stop search”

---

#### Spelling
Vim has an internal spell check (`:help spell`).
1. Set it with `:setlocal spell spelllang=en_us`
2. Run through the marked bad words with `[s` / `]s`
3. Use `zg` to mark a word as “good” (“add to dictionary”)
4. `zw` will mark an untagged word as “bad”

Some more options like file-local words for spelling, undo options etc can be found in the help section.

---

#### Insert mode quick editting
If you’re in insert mode and want to use a normal mode command without completely changing mode,
you can `Ctrl + o` which lets you send a single normal command without leaving.

Consider this:
I’m in insert mode and want to delete a word,
Option 1:
`esc` + `dw` + `i`
Option 2:
`Ctrl + o` + `dw`

30% less strokes :sl

---

### Working with [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe#goto-commands) like a boss
If you use [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe#goto-commands) plugin  it has plenty of extra features for different languages.
So now when I use TypeScript I can `./install.py --ts-completer` and beyond the completion there are hidden gems like GoTo definition or references, smart refactoring, organizing imports etc etc, here’s a short list of mappings I added:
nmap <leader>ygr :YcmCompleter GoToReferences<CR>
nmap <leader>ygt :YcmCompleter GoToType<CR>
nmap <leader>yGt :YcmCompleter GetType<CR>
nmap <leader>yGd :YcmCompleter GetDoc<CR>
nmap <leader>yfi :YcmCompleter FixIt<CR>
nmap <leader>yr :YcmCompleter RefactorRename<SPACE><C-R><C-W>
nmap <leader>yfm :YcmCompleter Format<CR>
nmap <leader>yo :YcmCompleter OrganizeImports<CR>
Docs are here: https://github.com/ycm-core/YouCompleteMe#goto-commands (edited)

---

### Getting to a specific charcter in a line
Use `|` preceeded with a number of a charter you wish to find.
Usefull when debugging and the debugger notifies of an error on line x and on char y, you can then use `:<x>` followed by `<y>|`.
A numerical example: "An error was found on line 12 character 3":
```vim
:12
3|
```

---

### Working with registers
`:reg` shows the current state of all registes and what is stored in them.
The numbered ones are a kind of a clipboard history showing the last 10 yanked / cut objects.

---

### Simple re-format for minified Javascript
command! UnMinify call UnMinify()
```vim
function! UnMinify()
  %s/{\ze[^\r\n]/{\r/g
  %s/){/) {/g
  %s/};\?\ze[^\r\n]/\0\r/g
  %s/;\ze[^\r\n]/;\r/g
  %s/[^\s]\zs[=&|]\+\ze[^\s]/ \0 /g
  normal ggVG=
endfunction
```

---

### Widescreen - bring code to 2/3 screen towards the middle
I found myself fighting a lot with vim windows when working on widescreens - I use vim on my left screen and the code naturally starts at the edge. I wanted a shortcut to “bring” the code closer to my eyes (that is - the right edge where my eyes rest). So I wrote this small function:
```vim
function! ComeCloser()
  :vnew<CR>
  :vertical resize 50
  :wincmd l
endfunc
nnoremap <C-w>x :call ComeCloser()<CR>
```

---

### Sessions
`:mks` will save your current session with all open tabs, for example I’m working on company.com project:
`:mks` company.vim
And next time I want to reopen the same state:
`vim -S mdi.vim`
[another tip: closing all tabs at once: `:qa`]

---

### Copy function below
Useful for duplicating a function for a similar structure and saves a few strokes
```vim
nmap <leader>yyb V}y}p<CR>
```

---

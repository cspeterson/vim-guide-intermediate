Vim guide - intermediate
=========

My slightly opinionated guide to Vim configuration and usage at an intermediate level.

There is some review of basics, but baseline competence is assumed.

# Contents

* [Vim Configuration](#vim-configuration)
	- [General Config](#general-config)
	- [Status Line](#status-line)
	- [Formatting Code](#formatting-code)
		- [ftdetect](#ftdetect)
		- [ftplugin](#ftplugin)
	- [Plugins](#plugins)
* [Vim Usage](#vim-usage)
	- [Modes](#modes)
	- [Navigation](#navigation)
	- [Undo and Redo](#undo-and-redo)
	- [Repeating an edit](#repeating-an-edit)
	- [Selecting text](#selecting-text)
		- [Visual mode](#visual-mode)
		- [Object selection](#object-selection)
	- [Deleting aka cutting](#deleting-aka-cutting)
	- [Yanking aka copying](#yanking-aka-copying)
	- [Motions](#motions)
	- [Pasting](#pasting)
	- [Normal mode commands](#normal-mode-commands)
	- [Indentation](#indentation)
	- [Splits](#splits)
		- [Splits settings](#splits-settings)
	- [Tabs](#tabs)
		- [Tabs settings](#tabs-settings)
	- [External commands and files](#external-commands-and-files)
* [Commandline tips and tricks](#commandline-tips-and-tricks)
* [My vimrc](#my-vimrc)


# Vim configuration

A walkthrough of the most important areas of configuration.

This walkthrough assumes the availability of a modern build of vim compiled with common features e.g. some of these things might not work on your sparse Vim build on RHEL 3.

For all of the detail not included here, you can also look at [The Official Vim Docs](http://vimdoc.sourceforge.net/) and [The Vim Wiki](http://vim.wikia.com/wiki/Vim_Tips_Wiki)

## General config

Here are some generally useful settings to consider.

To set or unset things in Vim, do these things from view mode with `:`, or put them directly into your .vimrc.

To investigate the current state of any setting, query it with a question mark, e.g. `:set tabstop?`

```viml
" line numbers on
set nu

" avoid old-style odd behaviours. things like word-skip and ctrl+some key combos  are probbaly kinda broken
set term=xterm-256color 

" always highlight the line being edited
set cursorline 

" Search with ignore case, smart case, highlight results, incrementally
" smart case ignores "ignore case" if we include uppercase characters
set ignorecase
set smartcase
set hlsearch
set incsearch
nnoremap <CR> :nohlsearch<cr> " enter clears search highlighting

" Disable modelines, a feature that lets text files have special vim commands in them, y u c k
set modelines=0

" Make visually-wrapped lines indent properly, but only if we're on a new enough version of vim
if has ( "patch-7.4.354" )
   set breakindent
endif
```

## Status line

You should always be able to tell where you are and what you are doing via the status line. But by default, it offers little information.

The status line is set via the setting `statusline`. Avoid illegibility by using concatenation.

```viml
" Here is an example statusline

" Always show statusline
set laststatus=2
" Statusline
set statusline=
" Buffer number
set statusline+=Buf:%(%{&filetype!='help'?bufnr('%'):''}\ \|\ %)
" Add full expanded path (without filename) of file, cut from right at 30 chars
set statusline+=%.30{fnamemodify(bufname('%'),':p:h')}/
" Add filename
set statusline+=%t
" Add modified or RO marker after filename if either true
set statusline+=%{&modified?'\ +\ ':''}
set statusline+=%{&readonly?'\ 🔒\ ':''}
" From here, align the rest to the right
set statusline+=%=
" File type as detected by vim, specifying when none
set statusline+=[%{&filetype!=#''?&filetype:'none'}]
" Show file encoding ig
set statusline+=%(\ \|%{(&bomb\|\|&fileencoding!~#'^$\\\|utf-8'?'\ '.&fileencoding.(&bomb?'-bom':''):'')\.(&fileformat!=#(has('win32')?'dos':'unix')?'\ '.&fileformat:'')}%)
" Show `et` or `noet` for expandtab on/off. Then, shift width
set statusline+=%(\ \|\ %{&modifiable?(&expandtab?'et\ ':'noet\ ').&shiftwidth:''}%)
" Column number
set statusline+=\ \|\ Col:\ %{&number?'':printf('%2d,',line('.'))}
set statusline+=%-2v " Virtual column number if differs
" Percentage through file
set statusline+=\ \|\ %2p%%
" Total lines
set statusline+=/%L
```

This config will result in a statusline like the following:
![Vim statusline](https://raw.githubusercontent.com/cspeterson/vim-guide-intermediate/master/imgs/statusline.gif)

## Formatting code

These are some settings most relevant to inputting code.

```viml
" Break lines on word separation at the value of `textwidth`. It is dumb: it just maintains the same indentation as
" the last line
set autoindent
" Draw a line in the editor at this column to indicate line limits (or whatever)
set colorcolumn=80
" This is generally presumed in a lot of places and you really might as well.
set encoding=utf-8
" Inserts n spaces instead of tabs where n=`tabstop`
set expandtab
" Windows linebreaks are gross and git often breaks when they are used
set fileformat=unix
" How many spaces an indentation will shift. This is distinct from tabstop in that it will generally only come
" into play when using << or >>
set shiftwidth=4
" Similar to `autoindent`, but will try to intelligently increase indentation for new code blocks. Can occasionally
cause issues when used alongside filetype-based indents.
set smartindent
" How many spaces to use instead of a tab when `expandtab` is set
set tabstop=4
" The legnth for line wrapping. Default is unset, so no line wrapping.
set textwidth=79
```

Of course, you are probably using a bunch of different languages with a bunch of different requirements. You can't just set these in your vimrc for everything and call it a day without creating new work for yourself. That is where filetype detectors(`:help ftdetect`) and filetype plugins (`:help ftplugin`) come in.

### ftdetect

An ftdetect script goes in `~/.vim/ftdetect/languagename.vim` and contains nothing but code to determine what language a file is. Oftentimes, it simply looks at the extension. Vim might already knows about your language though, and you mightn't need to write one of these.

But if you did, here is an example of recognition by file extension:

```viml
au BufRead,BufNewFile *.pp set filetype=puppet
```

The `filetype` set here will correspond with the name of the corresponding ftplugin, if there is one. In this case, it would be `puppet.vim`.

### ftplugin

An ftplugin script goes in `~/.vim/ftplugin/languagename.vim` and contains all the code and settings that you wish to apply to the particular language.

For instance, here is my [PEP 8](https://www.python.org/dev/peps/pep-0008/)-compliant ftplugin for python

```viml
set colorcolumn=80
set encoding=utf-8
set expandtab	" inserts spaces instead of tabs
set fileformat=unix
set shiftwidth=4
set smartindent
set softtabstop=4
set tabstop=4	" Sets a 'tab' to 4 spaces
set textwidth=79
```

## Plugins

So apart from basic configuration some folks have gone to the trouble of writing extensive, powerful plugins to extend Vim functionality or to assist in configuration for a particular language or platform.

The directory structure of a plugin is just a mirror of Vim's own, so you can manually "install" a plugin by copying all of the files into the right folders in your `~/.vim`. But that sucks - just use a plugin manager which can manage this for you.

[Vundle](https://github.com/VundleVim/Vundle.vim) is a popular plugin manager for Vim. [Pathogen](https://github.com/tpope/vim-pathogen) and [vim-plug](https://github.com/junegunn/vim-plug) are other options that I don't use.

Find useful Vim plugins by searching when you reach a pain point, or just browse around [VimAwesome](https://vimawesome.com/) and try out whatever strikes your fancy.

Vim plugins recommended by me in general:

* [Syntastic](https://github.com/vim-syntastic/syntastic) - syntax highlighting for like, everything
* [YouCompleteMe](https://github.com/Valloric/YouCompleteMe) - code completion for lots of stuff
* [NERDtree](https://github.com/scrooloose/nerdtree) - tree-based file browser

---

# Vim Usage

## Modes

* Normal - for navigation through the document and manipulating text. Hit `Escape` or `Ctrl+c` to get back here.
* Command-line - used to input Ex commands, searches, and filters. You hit `:` from Normal Mode to get here.
* Insert - for typing in text. Some commands from Normal mode will also work here. You can enter insert mode in a 
handful of different ways from normal mode:
   * `i` to begin insert mode at current cursor location
   * `a` to begin insert mode after the current cursor location
   * `A` to begin insert mode at the end of the line 
   * `I` to begin insert mode at beginning of text in current line
   * `o` to create a new line after the current line and begin editing there
   * `O` to create a new line before the current line and begin editing there
* Visual modes - There [several modes for text selection](#visual-mode) and manipulation. Most Normal mode commands work here.

## Navigation

In normal mode, cursor movement is the same between `h,j,k,l` and the arrow keys. You also have Page Up and Page Down at your disposal to for the big jumps.

## Undo and redo

* `u` undoes an action
* `ctrl+r` redoes an action

## Repeating actions

* `.` repeats the last edit action from normal mode. This is very useful when combined with object selection and 
replacement commands.
* `@:` repeats the last command in command-line mode.

## Selecting text

### Visual mode

Most operations *on* text need to have some text selected first. Use any of these from normal mode.

* `v` enters visual mode, where you can select text by moving the cursor
* `shift+v` enters visual *line* mode, where you can select text *by line*.
* `ctrl+v` enters visual *block* mode, where you can select rectangular chunks of text irrespective of line lengths.

Once the text you need is selected, you may yank or delete or otherwise operate upon it.

## Object selection

You can use object selections to select text at the object level. That is, word, sentence, paragraph, or block. This is a *really* great move to have under your belt for quick editing, code or otherwise.

* `iw` to select in a word
* `is` to select in a sentence
* `ip` to select in a paragraph
* `i'` to select in single quotes
* `i"` to select in double quotes
* `i{` or `i}` to select in curly braces
* `i[` or `i]` to select in brackets
* ``i` `` to select in backticks
* `i(` or `i)` to select in parens
* `i<` or `i>` to select in `<>` block

* `aw` to select  a  word (and its whitespace)
* `as` to select  a  sentence (and its whitespace)
* `ap` to select  a  paragraph (and its  whitespace)
* `a'` to select  a  single quoted string (and the quotes)
* `a"` to select  a  double quoted string (and the quotes)
* `a{` or `a}` to select block in curly braces (and the braces)
* `a[` or `a]` to select  block in brackets (and the brackets)
* ``a` `` to select texblock in backticks (and the backticks)
* `a(` or `a)` to select  block in  parens (and the parens)
* `a<` or `a>` to select  block in `<>` (and the `<>`)

There are more. `:help objects` has a lot more information.

Chain these up with `d` or `c` in Normal Mode to delete or change the value to be selected respectively.

* `d{object selection}` will "delete" (cut) the specified object and leave you back in Normal
* `c{object selection}` will do the same as the above, but will put you immediately into insert mode so you can replace
	the text.

## Deleting aka cutting

"Deleting" in normal mode in Vim cuts text into your pastebuffer.

* `d` cuts selected text, or can be followed by a vim motion
* `dd` cuts the current line

## Yanking aka copying

You can yank (copy) text a couple of ways in normal mode.

* `y` copies the selected text, or can be followed by a vim motion
* `yy` copies the current line

## Motions

Vim has "motions" for text operations in normal mode. `d` and `y` for delete and yank can be followed by a motion to delete or yank or just move the cursor in some distance in some direction. There are lots of motions, and you can dig into the full docs by running `:help motions` in normal mode. Here is a selection.

Left, down, up, and right here can all be substituted with h, j, k, and l if you're that sort of person.

Left/right motions

* `^` to the first *non-blank* character of the line
* `Home` to the first character of the line
* `$` or `End` to the end of the line
* `F[char]` back to but not on the last occurrence of character
* `f[char]` up to but not on the next occurrence of character
* `t[char]` up to and on the next occurence of character
* `T[char]` back to and on the last occurrence of character
* `[num]left|right` to [num] lines left or right

Up/down motions

* `[num]G` to [num] line. If [num] is not provided, defaults to bottom of file
* `gg` to top of file
* `GG` to bottom of file
* `[num]up|down` to [num] lines left or right
* `[num]%` to percentage in file by line count
* `:[num]` to line number

Word motions

* `ctrl+left|right` jump to beginning of word in direction

Motion repetition

* `;` repeat last `f`, `F`, `t`, or `T`
* `,` repeat last `f`, `F`, `t`, or `T` reversed in direction

## Pasting

### From within Vim itself

With something yanked or deleted, you can pop it back out again somewhere else as follows.

* `p` pastes text after the cursor
* `P` pastes text before the cursor
* `gp` pastes text after the cursor *and* moves the cursor to the end of the pasted text
* `gP` pastes text before the cursor *and* moves the cursor to the end of the pasted text

### From the system clipboard

To paste from the system clipboard in Insert Mode, `Ctrl+Shift+V`.

Vim has a bunch of settings re registers and clipboards and whatnot. These are beyond the scope of this document, but if you need to paste *code* into Vim there is something to consider: autoindent, if  on, can munge up indented code from the system clipboard. You'll want to turn on paste mode:

```viml
" Turn paste mode on
:set paste
" Turn paste mode off
:set nopaste
```

## Simple changes

Vim actually has a whole set of commands in [normal mode](#modes) just for making "simple changes." Here is a selection, and to learn more check the help `:help simple-change`.

* `~`, `u`, `U` toggle case, make lowercase, and make uppercase, respectively
* `Ctrl+a` increment the integer either at or next to the cursor.
	- The default is 1, but you can increment by e.g. 5 by instead typing `5Ctrl+a`. *Note: this may overlap with your GNU Screen escape key, in which case you can type `Ctrl+a+a` to send the correct keystrokes into Vim*
* `Ctrl+x` decrement an integer, and the size of the decrement can be specified as with increment above
* `r{char}` quickly replace the character under the cursor without leaving normal mode

## Indentation

Apart from the indentation *options* above regarding tabs and spaces and how they work, there are controls to shift lines and blocks of code around.

* `<<` shift line or selected text to the left
* `>>` shift line or selected text to the right
* `<iB` shift code *in block* to the left
* `>iB` shift code *in block* to the left
* `=i{` repair indentation of inner block excluding the braces
* `=a{` repair indentation of inner block including the braces

## Splits

You can split Vim into multiple windows within a session. This is really really really handy!

To open directly into splits via the commandline

```shell
# Open into N horizontal splits
vim -o[N]
# Open into N vertical splits
vim -O[N]
# Open list of files into horizontal splits
vim -o file1 file2 file3 ...
# Open list of files into vertical splits
vim -O file1 file2 file3 ...
```

And within Vim

```viml
" To split into a new or existing file
:sp newfilename
" Or to split into another view of the same file
:sp
" To do the same things, but with a vertical split, put a v in front
:vsp newfilename
:vsp
```

To move your focus between your splits, use `Ctrl+W`+`[left,right,up,down]`, or simply `ctrl+w w` repeatedly to jump through them in order.

To close all of the buffers *except* for the one you're currently using, use the command `:on[ly]` to keep "only" the current buffer.

To move a split window into a tab, `ctrl+w T`

### Splits settings

By default Vim puts new splits either on top or to the left of the current pane. You can change this behaviour by using the follow two settings:

```viml
set splitbelow " when splitting layout, new horizontal splits go below the current window
set splitright " when splitting layout, new vert splits go to the right of the current window
```

## Tabs

Vim can also use "tabs" which is more or less analagous to the function of a tabbed web browser. This allows sharing of registers within the Vim session, like splits, and might be a better organizational mechanism for your use case.

To open directly into tabs via the commandline

```shell
# Open into N tabs
vim -p[N]
# Open list of files into tabs
vim -p file1 file2 file3 ...
```

Within vim:

```viml
" To make a new tab
:tabedit [file]
```

To close all of the tabs *except* for the one you're currently using, use the command `:tabo[nly]` to keep "only" the current tab.

### Tabs settings

This may make it easier for you to work with tabs:

```viml
" In normal mode only, `Tab` to go forward through tabs, and `shift+Tab` to go backwards through tabs
nnoremap <S-Tab>   :tabprevious<CR>
nnoremap <Tab>     :tabnext<CR>
" `ctrl+t` to open a new tab
nnoremap <C-t>     :tabnew<CR>
```

## External commands and files

You can run external commands from Vim if there's something you need to know about the world outside your editor.

An example of what it can look like:

```viml
:!echo $RANDOM
8517

Press ENTER or type command to continue
```

You can also use this to pull the *output* of an external command directly into Vim.

```viml
" This will drop the output of the external command on the next line after your cursor.
:read !echo $RANDOM
```

Or this to directly  *read in* external files, as opposed to opening them in [splits](#splits) or [tabs](#tabs).

```viml
" This will drop in the contents of your fstab starting at the next line after the cursor
:read /etc/fstab
```

And this to pipe your file out though an external command and then back again
```viml
# Run current buffer through shfmt and then overwrite the current buffer with the result
%!shfmt
```

# [Commandline tips and tricks]

In addition to fluency *within* Vim itself, it may help to have lots of ways to get	*into* Vim.

``` sh
# Open two files into diff mode
vim -d file1 file2

# Open multiple files directly into horizontal splits
vim -o file1 file2

# Open multiple files directly into vertical  splits
vim -O file1 file2

# Open multiple files directly into tabs
vim -p file1 file2

# Open multiple files at once into Vim with horizontal splits, vertical splits, or tabs with find
find . -name '*somename*' -exec vim -o "{}" +
find . -name '*somename*' -exec vim -O "{}" +
find . -name '*somename*' -exec vim -p "{}" +
```

---
# My vimrc

If it helps, you can always see a live copy of my own Vim config in [my dotfiles](https://github.com/cspeterson/dotfiles).

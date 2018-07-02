compil-hints
============

Add _ballons_ and _signs_ to show where compilation errors have occurred.
The information is extracted from the [quickfix list](http://vimhelp.appspot.com/eval.txt.html#getqflist%28%29).


[![Last release](https://img.shields.io/github/tag/LucHermitte/vim-compil-hints.svg)](https://github.com/LucHermitte/vim-compil-hints/releases) [![Project Stats](https://www.openhub.net/p/21020/widgets/project_thin_badge.gif)](https://www.openhub.net/p/21020)

## Features
 * Parses the quickfix list and presents its entries as
   [_signs_](http://vimhelp.appspot.com/sign.txt.html#signs) and/or
   [_balloons_](http://vimhelp.appspot.com/debugger.txt.html#balloon%2deval) --
   when supported by Vim.
 * Works out of the box in synchronous and asynchronous compilation and
   grepping. IOW, asynchronous compilation plugins don't require to do anything
   to update the signs placed by compil-hints.
 * Multiple issues happening on a same line are merged together, the highest
   error level is kept (_error_ > _warning_ > _note_ > _context_), even when
   compilation/grep is asynchronous.
 * Balloons are automatically updated.
 * Signs are automatically updated at the end of a compilation, a (vim)grep...
   They are also automatically (and incrementally!) updated on asynchronous
   compilation/grepping -- that use
   [`:caddexpr`](http://vimhelp.appspot.com/quickfix.txt.html#%3acaddexpr),
   [`:grepadd`](http://vimhelp.appspot.com/quickfix.txt.html#%3agrepadd)...

 * Closing and opening the qf-window will activate and deactivate signs and
   balloons. IOW, when enabled, signs and balloons will be displayed only when
   the quickfix window is opened.
 * End-user can decide to globally use or disable the plugin, through the menu
   or the `auto_start` option.

 * Signs and balloons are updated on
   [`:cnewer`](http://vimhelp.appspot.com/quickfix.txt.html#%3acnewer) & al.

## Commands
Provides:
 * `:CompilHintsToggle` -- to start/stop using the plugin
 * `:CompilHintsUpdate` -- to update the signs to display; should not be
   required anymore

Listens/reacts on:
 * `:make`, `:grep`, `:vimgrep`, `:cscope`, `:cfile`, `:cgetfile`, `:helpgrep`, `:cexpr`, `:cgetexpr`, `:cbuffer`, `:cgetbuffer`
 * `:grepadd`, `:vimgreadd`, `:caddfile`, `:caddexpr`, `:caddbuffer`
 * `:copen`, `:cclose`, `:quit`, and anything that makes a qf window appears
 * `:cnewer`, `:colder`
 * `:call setqflist()`

 IOW, plugins that update the quickfix list don't need  to explicitly refresh
 the signs by calling `lh#compil_hints#update()` anymore since version 1.1.0.
## Demo

Here is a little screencast to see how things are displayed with vim-compil-hints.

![vim-compil-hints demo](doc/screencast-vim-compil-hints.gif "vim-compil-hints demo")

Note: You should observe that `:colder` suffers an important slow down in the
screencast (made on a slow VM). Since that screencast, I've improved the signs
unplacing execution by a 1200 times factor. It's likelly to still be slow when
all the buffers containining thousands of signs are loaded.

In order to workaround it, I could have set
[`g:compil_hints.harsh_signs_removal_enabled`](#bpgcompil_hintsharsh_signs_removal_enabled)
to 1. Alas, it will remove signs placed by other plugins as well.

## Options

The
[options](https://github.com/LucHermitte/lh-vim-lib/blob/master/doc/Options.md) are:

#### `g:compil_hints.use_balloons`
Activates the display of balloons -- boolean: [1]/0

Requires Vim to be compiled with
[`+balloon_eval`](http://vimhelp.appspot.com/various.txt.html#%2bballoon_eval)
support.

#### `g:compil_hints.use_signs`
Activates the display of signs -- boolean: [1]/0

Requires Vim to be compiled with
[`+signs`](http://vimhelp.appspot.com/various.txt.html#%2bsigns) support.

#### `g:compil_hints.autostart`
When sets, the plugin is automatically started -- boolean: 1/[0]

Needs to be set in the `.vimrc`.

#### `(bpg):compil_hints.context_re`
Regular expression used to recognize and display differently messages like:

```
instantiated from
within this context
required from here
```

#### `(bpg):compil_hints.harsh_signs_removal_enabled`
Improves greatly the time required to remove signs. However, this options does
remove all signs in a buffer, even the one not placed by compil-hints.

boolean: 1/0; default: `! exists('*execute')` => false with recent versions of
Vim

#### `g:compil_hints.signs`
Permits to specify which codepoints to use as sign characters depending on the
level:

- `error`   , defaults to `["\u274c", 'XX']`            -- '&#x274c;', 'XX'
- `warning'`, defaults to `["\u26a0", "\u26DB", '!!']`  -- '&#x26a0;', '&#x26db;', '!!'
- `note'`   , defaults to `["\u2139", "\U1F6C8", 'ii']` -- '&#x2139;', '&#x1f6c8;', 'ii'
- `context'`, defaults to `['>>']`
- `info'`   , defaults to `["\u27a9", '->']`            -- '&#x27a9;', '->'

This feature is used only when:
- [`'guifont'`](http://vimhelp.appspot.com/options.txt.html#%27guifont%27) is
  available (i.e. in graphical sessions),
- and when gvim doesn't support
  [`+xpm`](http://vimhelp.appspot.com/various.txt.html#%2bxpm) -- Pixmap
  support has the precedence.
- and when the Python module `python-config` can be used.

Otherwise, the last value in each list will be used. IOW, if you know that the
font you use in your terminal always support `\u274c`, you can simply define in
your [`.vimrc`](http://vimhelp.appspot.com/starting.txt.html#%2evimrc) (only!).

```vim
" Manually
:let g:compil_hints             = get(g:, 'compil_hints', {'signs': {}})
:let g:compil_hints.signs       = get(g:compil_hints, 'signs', {})
:let g:compil_hints.signs.error = ["\u274c"] " ❌


" Or, thanks to lh-vim-lib
call lh#dict#let(g:, 'compil_hints.signs.error', ["\u274c"]) " ❌

" Or, still thanks to lh-vim-lib
runtime plugin/let.vim
:LetTo g:compil_hints.signs.error = ["\u274c"] " ❌
```

Needs to be set in the `.vimrc`.

#### `g:compil_hints.hl`
Permits to specify which
[highlight](http://vimhelp.appspot.com/syntax.txt.html#%3ahighlight) group to
use depending on the level:

- `error`   , defaults to `"error"`
- `warning'`, defaults to `"todo"`
- `note'`   , defaults to `"comment"`
- `context'`, defaults to `"constant"`
- `info'`   , defaults to `"todo"`

Needs to be set in the `.vimrc`.

## Requirements / Installation

  * Requirements: Vim 7.2.295+,
    [`+balloon_eval`](http://vimhelp.appspot.com/various.txt.html#%2bballoon_eval),
    [`+signs`](http://vimhelp.appspot.com/various.txt.html#%2bsigns),
    [lh-vim-lib](http://github.com/LucHermitte/lh-vim-lib) 4.5.1.

  * With [vim-addon-manager](https://github.com/MarcWeber/vim-addon-manager), install vim-compil-hints.

    ```vim
    ActivateAddons vim-compil-hints
    ```

  * or with [vim-flavor](http://github.com/kana/vim-flavor) which also supports
    dependencies:

    ```
    flavor 'LucHermitte/vim-compil-hints'
    ```

  * or with Vundle/NeoBundle (expecting I haven't forgotten anything):

    ```vim
    Bundle 'LucHermitte/lh-vim-lib'
    Bundle 'LucHermitte/vim-compil-hints'
    ```

## TO DO
- Handle local options for balloon use: use/restore `b:bexpr`
- Ask fontconfig `fc-list`, when recent enough, which UTF-8 codepoints could be used -> lh-vim-lib
- Check the behaviour with encodings other than UTF-8.
- WIP: Permit to inject a different text to display in balloons (in grepping cases)
- Add a real option to inject `linehl` to signs
- Clean cached contexts from qf list no longer available with `c:older`
- When the qf-list isn't opened automatically at the end of the compilation,
  it's more tricky to remove the signs as `:cclose` doesn't do anything.
- Continue to improve the speed of `s:ReduceQFList` -- which is the slowest
  function of the plugin along with `s:WorksType`

## Notes and other implementation details
* It doesn't copy `getqflist()` for balloon, but always fetch the last version
  in order to automagically rely on vim to update the line numbers.


## History
* V 1.1.1
    * Improve sign placing and unplacing speed
* V 1.1.0.
    * Automatically activate the signs and balloons on quickfix related
      commands, whether the compilation is synchronous or asynchronous.
    * Improve style options
    * Distinguish _enabled_ and _activated_ states
    * Improve performances depending on whether the qf list is filled
      asynchronously or not
    * Balloons are filled differently quickfix list contain `grep` result
    * Keep track of context when navigating through qf history with `:cnewer` &
      all, with versions of vim recent enough (> v 7.4-2200)
* V 1.0.1.
    * Detect when XPM icons cannot be used.
    * Use the first UTF-8 glyphs
* V 1.0.0.
    * The XPM icons used come from Vim source code, they're under
      [Vim License](doc/uganda.txt).
    * Options have been renamed from `compil_hint_xxx` to `compil_hints.xxx`

* V 0.2.x.
    * This plugin is strongly inspired by syntastic, but it restricts its work to
    the result of the compilation.

---
description: NeoVim
---

# Vim

Installation

```bash
# Install vim
$ pip install neovim
# Install vim-plug
$ curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

Install Plugin

```text
$ mkdir ~/.config/nvim
$ nvim init.vim
```

```perl
cal plug#begin('~/.config/nvim/bundle')
Plug 'scrooloose/nerdtree'
Plug 'Shougo/deoplete.nvim', { 'do': ':UpdateRemotePlugins' }
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'tpope/vim-fugitive'
call plug#end()

syntax enable
set number
set cursorline
hi CursorLine term=bold cterm=bold guibg=Grey40

" vim airline config
let g:airline#extensions#tabline#enabled = 1
let g:airline#extensions#branch#enabled = 1
let g:airline_powerline_fonts = 1
let g:airline_theme='luna'

" deoplete config
let g:deoplete#enable_at_startup = 1

" map the <tab> to autocomplete
inoremap <expr> <Tab> pumvisible() ? "\<C-n>" : "\<Tab>"
inoremap <expr> <S-Tab> pumvisible() ? "\<C-p>" : "\<S-Tab>"

" config python for vim
let g:python3_host_prog = '/Users/canhtran/anaconda3/bin/python'

" nerdtree
map <C-n> :NERDTreeToggle<CR>
```

```bash
# Inside vim
:PlugInstall
```




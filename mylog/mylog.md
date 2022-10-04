## Technical Work Log

This document will be a log of my work. My work includes everything related to my tech career, e.g. my paid job, my personal projects, personal setup, freelance, .etc. My learning experience can appear here as well but I will set up another folders for storing my learning notes. 

### 2022-9-25

Today I have tried debugging vimtex by exploring its source code. The error I faced is "viewer cannot find Zathura window ID." I have no idea how I get this error at first. Because I have already installed Zathura on wsl, (plus a bunch of other sofwares, e.g. vcsxrv, x11-apps) and the zathura is working on my wsl. I can open a pdf window in wsl terminal to view documents in windows. So it must not be related to Zathura installation problem. 

I have tried changing the setting: delete the default g:vimtex_view_method="zathura" and change to g:vimtex_view_general_viewer="zathura" in init.lua instead. It works. I can now use VimtexView to see the pdf file compiled. But forward search and inverse search don't work. 

That' why I decided to inspect the source code of vimtex and see the line that calls the zathura command. After inspecting the function and learning about vimscript language, I locate a self._start method and insepct the function called inside this method. After 1-2 hours, I found the line calling the shell command of zathura and pdf file, which is:

```vim
function! s:cmdline(outfile, synctex, start) abort " {{{1
  let l:cmd  = 'zathura'

  if a:start
	let l:cmd .= ' ' . g:vimtex_view_zathura_options
	if a:synctex
	  let l:cmd .= printf(
			\ ' -x "%s -c \"VimtexInverseSearch %%{line} ''%%{input}''\""',
			\ s:inverse_search_cmd)
	endif
  endif

  if a:synctex && (!a:start || g:vimtex_view_forward_search_on_start)
	let l:cmd .= printf(
		  \ ' --synctex-forward %d:%d:%s',
		  \ line('.'), col('.'),
		  \ vimtex#util#shellescape(expand('%:p')))
  endif

  return l:cmd . ' '
		\ . vimtex#util#shellescape(vimtex#paths#relative(a:outfile, getcwd()))
		\ . '&'
endfunction	
```
So, the problem lies in the flag argument passed to the zathura command when opening pdf. I have tried to use the same flag:

	zathura --synctex-forward 28:1:hello.tex hello.pdf

It gives an error of course. Bingo! Now the error probably springs from this command. So i went on to google this error and the solution is install dbus-x11. Go ahead and all errors are gone. 


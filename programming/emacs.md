# Emacs

## Using Emacs to read C code

[Create etags table](https://www.gnu.org/software/emacs/manual/html_node/emacs/Create-Tags-Table.html) to store symbols definitions and references:
```
find . -name "*.[chCH]" -print | etags -
```

Useful commands to navigate the code:

- `M-x rgrep`: To search in files of source code
- `M-.` (`xref-find-definitions`): (Require etags) Use it over a function or
  variable reference to find the definition. If we use it over no symbol, it ask
  for the definition to search.
- `M-?` (`xref-find-references`): (Require etags) Find references to a given
  symbol.
- `M-,` (`xref-pop-marker-stack`) : (Require etags) Go back in the symbol search
  stack.
  
- `C-M-.` (`xref-find-apropos`): (Require etags) Find a symbol based on a
  regex.


To convert Emacs in a more complete C IDE: [C/C++ Development Environment for Emacs](https://tuhdo.github.io/c-ide.html)

## General Tips

### Open file as root

To opening file as root or sudo, prepende the file path with `/su::` or
`/sudo::`. For example:
```
C-x C-f /su::/etc/shadow/~
```

### Open file over ssh

```
C-x C-f /ssh:user@192.168.0.24:file.txt
```

Resource:
- [What is the best way to open remote files with emacs and ssh](https://stackoverflow.com/a/20624538)

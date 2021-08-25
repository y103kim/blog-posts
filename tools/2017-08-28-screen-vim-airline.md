---
layout: post
category: tools
title: screen에서 vim-airline을 사용하기 위한 설정
modified: 2017-08-28
tags: [vim, screen, vim-airline]
---

## putty 설정

DejaVu Sans Mono for Powerline 으로 폰트 설정하자. Font Quaility는 Cleartype으로 설정했다.

## vimrc 설정

가장 중요한 설정은 터미널 설정이다. 

airline에서 Powerline 스타일의 삼각형 폰트를 보고싶다면, powerline 폰트 옵션을 켜야 한다. 또한 적절한 테마를 설정하고, color옵션도 켜주어야 한다.

```
set term=xterm-256color
set t_Co=256
let g:airline_theme="wombat"
let g:airline_powerline_fonts = 1
```

## screenrc 설정하기

정말로 많은 삽질과 구글링을 반복해 얻은 정보이다. 해당 구문을 screenrc에 삽입하면 putty기준 깨짐 없이 모든것이 잘 보인다.

```
attrcolor b ".I"    # allow bold colors - necessary for some reason
defbce on    # use current bg color for erased chars
termcapinfo xterm* 'is=\E[r\E[m\E[2J\E[H\E[?7h\E[?1;4;6l'
```

## 추가로 설정하기

vim airline 사용중에 몇가지 유용한 옵션이 있다.

```
let g:airline#extensions#tabline#enabled = 1
let g:airline#extensions#tabline#fnamemod = ':t'
```

첫 번째 옵션은 가장 윗 줄에 tabline이 보이게 하는 옵션이다. 탭이 하나일 때는 버퍼가 나열된다.
두 번째 옵션은 tabline에 파일 이름만 보이게 하는 옵션이다. 복잡하게 경로가 뜨지 않는다.
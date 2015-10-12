---
layout: post
title: Vim 기반의 Markdown 문서 작성
tags: vim,markdown
---

지극히 개인적인 취향의 vim에서 마크다운 문서 작성하는 환경을 설명한다.
일반적인 마크다운 사용법은 [놀부의 마크다운 사용법](http://nolboo.github.io/blog/2014/04/15/how-to-use-markdown/)을 참조하길 바란다.


외부적인 요구가 없는 한 대부분의 문서를 나는 마크다운으로 작성하고 있다.
- 프로젝트 전/후의 문서화
- 블로그
- 아이디어 정리 메모
- 작업중 기억이 필요한 메모(DayOne 앱 사용)


자료 수집/정리보다는 나의 생각/아이디어를 정리하는 노트 도구로써 마크다운 사용법을 정리/설명 한다. Max OSX 환경에서의 방법이며 Windows 사용자에게는 도움이 되지 않을 수 있다.


## 나의 요구사항

### 생산성

1. Vim keymap!!!
1. Live Preview: 파일이 변경되면 자동으로 미리보기가 가능해야한다.

### 표현력

1. [GitHub Flavored Markdown](https://help.github.com/articles/github-flavored-markdown/)
    - S/W 개발관련 주제가 대부분이므로 Sytax Highlight는 필수
    - 테이블
1. 순서도, 시퀀스다이어그램
    - 한장의 다이어그램이 백마디보다 낫다.
    - 빠르게 생성할 수 있는 단순한 이미지면 충분하다.


## 문서 작성 Workflow

![]({{ site.baseurl }}/assets/2015/markdown-workflow.mmd.png)


### Vim - Markdown Editor

마크다운 문법 강조 기능을 추가하고 문서를 저장할 때마다 브라우저가 자동 새로고침되면서 미리보기할 수 있도록 Vim을 마크다운 편집기로 강화시켜보자.

vim 7.4에는 마크다운 문법 강조가 내장되어있지만 GFM을 지원하기 위해서 [vim-flavored-markdown](https://github.com/jtratner/vim-flavored-markdown) 플러그인을 추가 설치한다. 설치 방법은 vim 플러그인 매니저마다 차이가 있지만 나는 [neobundle](https://github.com/Shougo/neobundle.vim)을 사용하고 있고 아래 처럼 플러그인을 추가할 수 있다.

```vim
NeoBundle "jtratner/vim-flavored-markdown"
```

대부분의 마크다운 전용 편집기는 실시간 미리보기 기능을 포함하고 있다. Vim에서는 추가 플러그인 설치로 가능하지만 나의 경우엔 Node 모듈인 [livedown](https://github.com/shime/livedown)을 Tmux와 연동해 사용한다. 물론 [livedown vim plugin](https://github.com/shime/vim-livedown)을 이용할 수도 있다. livedown을 선택한 이유는 다은 방법에 비해서 미리보기 지연시간이 가장 적기때문이다.

```vim
nnoremap <leader>mp :!tmux split-window -l 5 "livedown start %:p --verbose"<CR>
```

- Normal 모드에서 `<leader>mp`키를 누르면 `tmux split-window -l 5` 5줄 높이의 tmux 윈도우를 하단에 연단.
- `livdown start %:p` 편집중인 버퍼의 경로를 livedown에 파라미터로 주고 실행한다. 기본 브라우저가 열리고 http://localhost:1337 URL에서 미리보기를 볼 수 있다.

이제 부터는 파일을 저장할 때 마다 변경된 내용이 자동으로 브라우저에 보여진다.

![vim-tmux-livedown](https://www.evernote.com/l/AAL4AOOq_pZLpqmtCrSFldVGcf8wYD5trH8B/image.png)


### Mermaid - 다이어그램 생성

[하루패드](http://pad.haroopress.com/)와 같은 마크다운 전용에디터는 간단한 [텍스트로 표현하는 다이어그램](http://pad.haroopress.com/page.html?f=how-to-draw-diagram)을 지원한다. [mermaid](https://github.com/knsv/mermaid)를 통해서 지원되는 기능인데 코드 블록내에 mermaid 문법을 기입하고 memaid.js를 통해 동적으로 변환하는 방식이다. 동적 mermaid 처리 외에는 Haroopad는 내가 원하는 모든 기능을 지원하는 괜찮은 에디터이다.

마크다운 문서를 배포하는 환경이 mermaid를 지원하기 어려운 경우가 많으므로 나의 경우에 마크다운 문서와 같은 위치에 `.mmd` 파일을 만들고 vim에서 `.mmd` 파일을 저장하면 [memaid CLI](http://knsv.github.io/mermaid/#mermaid-cli)를 통해서 이미지 파일을 생성하도록 하고 이 이미지 파일을 markdown 문서에 이미지 url로 추가한다.

위에서 본 다이어그램 역시 같은 방식으로 작성하였다. 아래는 `.mmd` 파일의 내용이다.

```
graph LR
    vim((Vim + livedown plugin));
    md[.md 파일];
    mmd[.mmd 파일];
    png[.png 이미지파일];
    chrome((localhot:1337));

    vim-->md;
    vim-->mmd;
    mmd-->png;
    png-->md;
    md--Preview, Refresh-->chrome;
```

#### mermaid 설치

```
npm -g install phantomjs mermaid
```

사용중인 mermaid 버전은 `0.5.5`인데 최신의 Node 버전인 `4.1.2`에서는 mermaid 모듈 설치시 문제가 발생한다. 그래서 markdown 문서를 작성할 때는 Node 0.12.7을 사용한다.

maermaid CLI는 이미지 출력을 위해서 phantomjs에 의존한다. phantomjs 모듈도 설치해야 한다.

#### vim autocmd 등록

```vim
autocmd BufWritePost *.mmd !mermaid -t /usr/local/lib/node_modules/mermaid/dist/mermaid.forest.css '%:p' -o '%:h' -p -v
```

- `autocmd BufWritePost *.mmd !mermaid` 확장자가 mmd인 파일을 저장하면 mermaid 외부 명령을 실행한다.
- `-t /usr/local/lib/node_modules/mermaid/dist/mermaid.forest.css` mermaid는 `mermaid.css`와 `mermaid.forest.css` 2개의 theme을 포함하고 있는데 그 중 forest 테마를 사용한다.
- `'%:p'` 현재 버퍼의 절대경로
- `-o '%:h'` 출력되는 이미지를 저장할 경로, 현재 버퍼의 경로(파일이름 제외)
- `-p` png 파일 출력

> 레티나 기기를 사용한다면 출력되는 png 파일이 너무 작을 수 있는데, 이 때는 `/usr/local/lib/node_modules/mermaid/lib/phantomscript.js` 파일의 265~266줄을 수정하여 이미지 크기를 키울 수 있다. Node 모듈을 직접 수정하는 것은 좋지 못한 것 같으나 다른 해결법을 찾지 못했다.

```diff
-width = boundingBox.width * 1.5; // adding the scale factor for consistency with output in chrome browser
-height = boundingBox.height * 1.5; // adding the scale factor for consistency with output in chrome browser
+width = boundingBox.width * 3; // adding the scale factor for consistency with output in chrome browser
+height = boundingBox.height * 3; // adding the scale factor for consistency with output in chrome browser
```


## 추가

글쓰기에 방해받지 받고 집중력을 높일 수 있도록 도와주는 Vim 플러그인 [고요](https://github.com/junegunn/goyo.vim)


## 여러가지 Markdown Editor

- marxi.co
    - stackedit 와 유사
    - evernote 연동이 장점. 유료

- stackedit.io
    - marxi.co와 동일한 문법
    - dropbox, google drive 연동.

- haroopad
    - mermaid 문법


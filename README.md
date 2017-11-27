# 자바카페 블로그 운영

* https://github.com/JAVACAFE-STUDY/official-blog.git clone 후, **_posts** 하위 폴더에 마크다운 파일 업로드해주시면 됩니다.
* 파일 이름의 형식은 날짜-제목 형식으로 하면되고, 확장자는 md로 작성해주세요.
  * 예) 2017-04-08-자바스크립트-패턴-1주.md
  * '-' 의 경우 자동적으로 띄어쓰기로 변경 됩니다.
* 이미지의 경우, img 폴더 하위에 넣어주세요
  * commit 후 push 하면 블로그 사이트에 자동 배포 됩니다. (혹시 push 권한 없으면 밴드에 문의해주세요.)
* 확인 : [http://tech.javacafe.io/](http://tech.javacafe.io/)

![guide1](/img/javacafe/guide1.png)



* 참고 : 마크다운 작성시, Typora 라는 프로그램을 사용할 수 있습니다.
  * https://typora.io/

---

# Local 에서 jekyll 로 블로그 서버 확인방법

현재 홈페이지는 jekyll 로 실행되고 있으며 홈페이지 자체 레이아웃 변경 테스트를 위해서는 Local 컴퓨터에서 jekyll 로 확인하는 것이 좋다.

1. jekyll 을 설치하기 위해서는 ruby 환경이 필요하다. 루비부터 설치하자. 노드제이에스도 설치하자.

2. 의존성 있는 gem 모듈 설치 

```bash
gem install bundler
gem install jekyll-paginate
gem install jekyll-paginate
gem jekyll-feed
```

3. 의존성 있는 노드 모듈 설치

```bash
npm install
```

3. jekyll 로 서버동작 시키기 

```bash
jekyll serve -w
```

4. _config.yml 변경 후, 클린 및 빌드

```bash
jekyll clean
jekyll build
```

---


# Clean Blog by Start Bootstrap - Jekyll Version

The official Jekyll version of the Clean Blog theme by [Start Bootstrap](http://startbootstrap.com/).

### [View Live Demo &rarr;](http://blackrockdigital.github.io/startbootstrap-clean-blog-jekyll/)

## Before You Begin

In the _config.yml file, the base URL is set to /startbootstrap-clean-blog-jekyll which is this themes gh-pages preview. It's recommended that you remove the base URL before working with this theme locally!

It should look like this:
`baseurl: ""`

## What's Included

A full Jekyll environment is included with this theme. If you have Jekyll installed, simply run `jekyll serve` in your command line and preview the build in your browser. You can use `jekyll serve --watch` to watch for changes in the source files as well.

A Grunt environment is also included. There are a number of tasks it performs like minification of the JavaScript, compiling of the LESS files, adding banners to apply the MIT license, and watching for changes. Run the grunt default task by entering `grunt` into your command line which will build the files. You can use `grunt watch` if you are working on the JavaScript or the LESS.

You can run `jekyll serve --watch` and `grunt watch` at the same time to watch for changes and then build them all at once.

## Support

Visit Clean Blog's template overview page on Start Bootstrap at http://startbootstrap.com/template-overviews/clean-blog/ and leave a comment, email feedback@startbootstrap.com, or open an issue here on GitHub for support.

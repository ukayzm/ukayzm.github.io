---
title:  "블로그 운영 - references"
categories: blog jekyll
layout: post
---

이 블로그는 Jekyll로 만들어져 있습니다.

## Install Jekyll

```
$ sudo apt-get install ruby ruby-dev build-essential
$ echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
$ echo 'export GEM_HOME=$HOME/gems' >> ~/.bashrc
$ echo 'export PATH=$HOME/gems/bin:$PATH' >> ~/.bashrc
$ source ~/.bashrc
$ gem update
$ sudo gem update --system
$ gem install jekyll bundler
```

## Upgrade Jekyll
```
$ jekyll --version
$ gem list jekyll
$ bundle update jekyll
$ gem update jekyll
```

## Run Jekyll
```
$ jekyll serve --draft --host 0.0.0.0
```

## References

* [Jekyll Official Docs][jekyll-docs]
* [Atom 을 마크다운(Markdown) 에디터로 사용하기][atom-guide]
* [Mastering Markdown](https://guides.github.com/features/mastering-markdown/)
* [Jekyll Blog 추천 테마 5가지][jekyll-theme-recommendation]
* [Jekyll 블로그 테마 적용하기 (minimal-mistakes)][jekyll-minimal-mistakes]
* [Google Analytics](https://analytics.google.com/)
* [구글 애널리틱스 설치부터 적용까지](https://milooy.wordpress.com/2016/01/14/google-analtyics-1-intro/)

  * Minimal-mistakes 테마를 사용하는 경우에는, _config.yml 파일을 아래와 같이 수정하면 자동으로 google analytics 스크립트가 붙는다.

```
...
# Analytics
analytics:
  provider               : google-gtag # false (default), "google", "google-universal", "google-gtag", "custom"
  google:
    tracking_id          : UA-111111111-1
...
```

### Type-on-Strap theme

* [http://jekyllthemes.org/themes/Type-on-Strap/](http://jekyllthemes.org/themes/Type-on-Strap/)
* Adsense를 위해 _include에 adsense.html을 추가하고 _include/head.html의 마지막 줄을 아래와 같이 수정

```
...
{% raw %} {% include adsense.html %} {% endraw %}
</head>
```

* URL을 /:categories/:title/ 형태로 만들기 위해 _config.yml 파일에 아래와 같이 permalink 라인을 추가

```
...
# PAGINATION
permalink: /:categories/:title/
paginate: 5
paginate_path: "blog/page:num"
...
```


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[jekyll-theme-recommendation]: https://isme2n.github.io/devlog/2017/03/09/Blog-Jekyll-theme/
[atom-guide]:  http://futurecreator.github.io/2016/06/14/atom-as-markdown-editor/
[jekyll-minimal-mistakes]: https://junhobaik.github.io/jekyll-apply-theme/

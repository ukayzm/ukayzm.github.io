---
title:  "블로그 운영 - references"
categories: blog jekyll
header:
  teaser: /assets/img/pexels/blog-blocks-wallpaper-1591056.jpg
  image: /assets/img/pexels/blog-blocks-wallpaper-1591056.jpg
date:   2018-03-16 20:00:00 +0900
toc: true
toc_sticky: true
sidebar:
  - title: "블로그 운영"
    image: /assets/img/pexels/wall-e-toy-on-beige-pad.jpg
    image_alt: "image"
    text: "references"
---

이 블로그는 Jekyll로 만들어져 있습니다.

# Install Jekyll

```
$ sudo apt-get install ruby ruby-dev build-essential
$ echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
$ echo 'export GEM_HOME=$HOME/gems' >> ~/.bashrc
$ echo 'export PATH=$HOME/gems/bin:$PATH' >> ~/.bashrc
$ source ~/.bashrc
$ gem update
$ gem install jekyll bundler
$ bundle update
```

# Upgrade Jekyll
```
$ jekyll --version
$ gem list jekyll
$ bundle update jekyll
$ gem update jekyll
```

# Run Jekyll
```
$ jekyll serve --draft --host 0.0.0.0
```

# References

* [Jekyll Official Docs][jekyll-docs]
* [Atom 을 마크다운(Markdown) 에디터로 사용하기][atom-guide]
* [Mastering Markdown](https://guides.github.com/features/mastering-markdown/)
* [Jekyll Blog 추천 테마 5가지][jekyll-theme-recommendation]
* [Jekyll 블로그 테마 적용하기 (minimal-mistakes)][jekyll-minimal-mistakes]
* [Google Analytics](https://analytics.google.com/)
* [구글 애널리틱스 설치부터 적용까지](https://milooy.wordpress.com/2016/01/14/google-analtyics-1-intro/)
* [minimal-miatakes layout](https://mmistakes.github.io/minimal-mistakes/docs/layouts/)
* [Pexels - free image](https://www.pexels.com/)
* [Font Awesome - free icons](https://fontawesome.com/icons?d=gallery)

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

# Type-on-Strap theme

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

# minimal-mistakes theme

```bash
git clone https://github.com/mmistakes/minimal-mistakes.git YourDirectory
cd YourDirectory
rm -rf .editorconfig .gitattributes .github docs/ test/ CHANGELOG.md minimal-mistakes-jekyll.gemspec README.md screenshot-layouts.png screenshot.png    # remove unnecessary files
cat << EOF > Gemfile     # overwrite Gemfile
source "https://rubygems.org"

# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#     bundle exec jekyll serve
#
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!

# gem "github-pages", group: :jekyll_plugins

# To upgrade, run `bundle update`.

gem "jekyll"
gem "minimal-mistakes-jekyll"

# The following plugins are automatically loaded by the theme-gem:
#   gem "jekyll-paginate"
#   gem "jekyll-sitemap"
#   gem "jekyll-gist"
#   gem "jekyll-feed"
#   gem "jekyll-include-cache"
#
# If you have any other plugins, put them here!
group :jekyll_plugins do
end
EOF
bundle install     # install gems then Gemfile.lock is created
```

# Page Redirect (jekyll-redirect-fromn)

Install.

```bash
$ vi Gemfile   # add this line
gem "jekyll-redirect-from"
$ bundle
```

Add the plugin jekyll-redirect-from to your `_config.yaml` file.

```
plugins:
  - jekyll-redirect-from

# mimic GitHub Pages with --safe
whitelist:
  - jekyll-redirect-from
^D
```

## Redirect Internally

Add this to your file’s frontmatter.

```
---
title: Your Post
redirect_from:
  - /old/page/
  - /old2/page2
---
```

That will create a /old/page/index.html and an /old2/page2 page that redirects to the Your Post page.

## Redirect to Other Sites

```
---
title: Your Post2
redirect_to:
  - https://www.dest.com/
---
```

That will create a page for Your Post2 to redirect to https://www.supertechcrew.com/.

# Search Engine

주요 서치 엔진에 블로그를 등록하면 다른 사람들이 서치를 할 수 있다.

## Google

[Google Search Console](https://search.google.com/search-console/about)에서 등록한다.

## Naver

[네이버 서치 어드바이저](https://searchadvisor.naver.com/)에서 등록한다.

상단의 "웹마스터 도구"를 누르고, 사이트를 입력.

## Daum

[다음 검색 등록](https://register.search.daum.net/index.daum)에서 등록한다.

# Tips

Notice<br>\{: .notice\}
{: .notice}

Notice info<br>\{: .notice--info\}
{: .notice--info}

Notice Warning<br>\{: .notice--warning\}
{: .notice--warning}

Notice Danger<br>\{: .notice--danger\}
{: .notice--danger}

Notice Success<br>\{: .notice--success\}
{: .notice--success}

<figure>
    <a href="/assets/img/pexels/wall_e.jpeg" class="align-center"><img src="/assets/img/pexels/wall_e.jpeg"></a>
    <figcaption>Image click to enlarge (this is caption)</figcaption>
</figure>


Center aligned text and image<br>\{: .text-center\}
{: .text-center}

![image in center](/assets/img/triangle.svg){: .align-center}
\!\[image in center\]\(/assets/img/triangle.svg\)\{: .align-center\}
{: .text-center}


# MacOS에서 설치

Ruby를 설치한다.

```zsh
$ brew install ruby
$ ruby -v
ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.arm64e-darwin20]
```

Ruby 버전 관리를 위해서 rbenv를 설치한다.

```zsh
$ brew update
$ brew install rbenv ruby-build  # rbenv, ruby-build 설치
$ eval "$(rbenv init -)"         # .bashrc나 .zshrc에 이 라인을 추가한다.
```

rbenv를 이용하여 원하는 버전의 ruby를 설치한다. (이 글을 쓰는 시점에서의 최신 버전은 2.7.2)
```zsh
$ rbenv install -l    # 설치 가능한 최신 버전의 ruby를 확인
$ rbenv install 2.7.2
$ rbenv rehash        # 새 Ruby 설치 후 실행합니다.
```

사용할 ruby의 버전을 지정한다.

```zsh
$ rbenv global 2.4.4 # 시스템 전역의 버전을 지정
$ rbenv local 2.3.2  # 현재 디렉토리에서 사용할 버전 지정
$ ruby -v
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [arm64-darwin20]
```

bundle로 gem을 설치한다.

```zsh
$ bundle install
```

# 참고사이트

[https://devinlife.com/howto%20github%20pages/google-search-console-and-analytics/](https://devinlife.com/howto%20github%20pages/google-search-console-and-analytics/)
[https://devinlife.com/howto%20github%20pages/register-search-engine/](https://devinlife.com/howto%20github%20pages/register-search-engine/)


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[jekyll-theme-recommendation]: https://isme2n.github.io/devlog/2017/03/09/Blog-Jekyll-theme/
[atom-guide]:  http://futurecreator.github.io/2016/06/14/atom-as-markdown-editor/
[jekyll-minimal-mistakes]: https://junhobaik.github.io/jekyll-apply-theme/

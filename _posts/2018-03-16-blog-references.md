---
title:  "블로그 운영 - bookmarks"
categories: blog jekyll
---

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
# Analytics
analytics:
  provider               : google-gtag # false (default), "google", "google-universal", "google-gtag", "custom"
  google:
    tracking_id          : UA-111111111-1
```


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[jekyll-theme-recommendation]: https://isme2n.github.io/devlog/2017/03/09/Blog-Jekyll-theme/
[atom-guide]:  http://futurecreator.github.io/2016/06/14/atom-as-markdown-editor/
[jekyll-minimal-mistakes]: https://junhobaik.github.io/jekyll-apply-theme/

# My first blog.

## hugo를 사용

> plan 카테고리 신규 생성

```
hugo new --kind plan post/plan/title.md
```

## hugo local 실행

```
hugo serve -D
```

## 컨텐츠 업로드

> 테마가 적용된 블로그 내용을 public에 생성하기.
```
문법] hugo -t <테마이름>

실행] hugo -t tranquilpeak
```

> public 디렉토리 이동 후, add, commit, push

```
cd public
git add .
git commit -m "커밋메시지" .
git push origin master
```

> blog 도 변경내용 push 하기
```
git add .
git commit -m "커밋메시지" .
git push origin master
```

## remote와 Submodule 설정

> blog remote 연결
```
git remote add origin git@github.com:hyunwoo-kr/blog.git
```

> submodule 연결

```
git submodule add -b master git@github.com:hyunwoo-kr/hyunwoo-kr.github.io.git public
```

## git submodule 삭제방법

> 모듈 deinit 하기.

```
git submodule deinit -f public
```

> .git/modules/public 삭제

```
rm -rf .git/modules/public
```

> 해당 폴더 삭제

```
git rm -f public
```
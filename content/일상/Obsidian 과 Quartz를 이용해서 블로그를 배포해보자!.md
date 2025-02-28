---
aliases:
  - "\bQ"
---
블로그를 운영해야지.. 운영해야지.. 하면서 제대로 운영을 안하다가 이번 글쓰기 스터디를 통해 운영을 시작해보기로 했다.

그러던 중 평소 Notion, Obsidian과 같은 툴을 이용하면서 공부했던 내용을 하는데 이를 정리해서 Velog, Tistory 같은 블로그에 올리려니 너무 귀찮다는 생각이 들었다.

그렇다고 화면을 구성하지 못하는 범부인 백엔드 개발자인 나는 이번 기회에 Obsidian과 Quartz를 이용해서 블로그를 배포해보기로 했다.

## Obsidian

![[Pasted image 20250301031929.png]]

 > Obsidian은 마크다운 파일에서 작동하는 메모 작성 어플리케이션이다.
 

실제 앱을 다운 받아서 사용하면 아래와 같이 구성된다. (실제로 Extension들도 제공하고 있어 Notion과 비슷하게 사용 가능하다. )

![[Pasted image 20250301032036.png]]


## Quartz

![[Pasted image 20250301032353.png]]


> github에서 제공하는 마크다운 기반 웹 사이트를 생성하게 해주는 도구

위 사진에서 user this template을 통해 내 깃허브에 레포지토리를 생성해준다.

![[Pasted image 20250301032921.png]]


이렇게 생성된 레포지토리를 나의 로컬 환경에 `git clone` 한 후 이를 다음과 같이 Obsidian에서 가져온다.

![[Pasted image 20250301033546.png]]
[blog 위치]


![[Pasted image 20250301033616.png]]
[Obsidian 화면]

위 화면에서 보관함 폴더 열기를 통해 내가 복제한 폴더를 가져온다.


![[Pasted image 20250301033746.png]]

이후 화면 뜨면 index 부분이 메인?화면 이기에 아무 말이나 써준다.
(이때 content 폴더에 글들을 작성해야 한다!)

작성한 후에는 개발자들이 매일 하는 git commit, push를 통해 github repository에 적용해줍니다!

## Cloudflare

> CDN 서비스와 분산 네임서버를 이용하여 사이트를 생성하게 해주는 하나의 플랫폼


![[Pasted image 20250301034025.png]]

이제 github에 올려둔 repository를 연동하여 무료로 배포를 하기 위해 Cloudflare를 사용할 것이다.(배포 방법은 다양하게 있으니 원하는 방식으로 해도 괜찮을 것 같다.)

해당 부분에 들어와서 Workers 및 Pages에서 생성버튼을 누르면 다음과 같은 화면이 뜬다.

![[Pasted image 20250301034436.png]]

여기서 내가 블로그글을 저장하고 있는 `blog` Repository를 가져온다. 

![[Pasted image 20250301034710.png]]

이후 이와 같이 작성하여 조금 기다려주면 아래와 같이 블로그를 배포할 수 있다.

![[Pasted image 20250301034800.png]]


아무래도 기본적인 것만 가져왔기에 좀 이쁘지는 않고 다른 블로그들에 부족한 점이 많아 보이는데 
`https://quartz.jzhao.xyz/` 해당 사이트에서 좀 더 구성할만한 부분, 플러그인들을 찾아보고 추후에는 내 도메인을 직접 생성하여 적용해 볼 예정이다.

+그리고 매번 git commit, push.. 하는 게 생각보다 너무 귀찮다... FlowerShop같은 플러그인을 사용하기도 하던데 도메인 설정할 즈음에 같이 수정해봐야겠다.
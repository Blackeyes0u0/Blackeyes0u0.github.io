[Github 블로그] 블로그 포스팅하는 방법
 Date: 2020.05.25    Updated: 2020.05.25
 카테고리: Blog

 태그: Github Git jekyll Blog

 목차
1. Markdown 을 지원하는 에디터를 실행한다.
2. _posts 폴더를 생성한다.
3. yyyy-mm-dd-title.md 형식의 파일을 만들어준다.
4. 머릿말(Front-Matter)을 상단에 작성해 주어야 한다.
5. 포스트 내용을 Markdown 문법으로 작성한다.
6. 마크다운으로 쓴 포스트 내용을 미리 보기
Markdown Preview Enhanced
로컬 서버에서 블로그에 적용될 모습 확인하기
7. 작성한 포스트 파일을 git push 하여 Github Pages 서버에 업로드 한다.
8. 다른 컴퓨터에서 포스팅 작업을 하고 싶을 때 git pull
1. Markdown 을 지원하는 에디터를 실행한다.Permalink
나는 Visual Studio Code 에디터를 선택했다. 집에 Atom이 깔려 있어 처음에는 Atom 으로 첫 포스팅을 했었으나 묘하게 불편했다. 프리뷰 기능을 키면 렉인지 뭔지 보여졌다 안 보여졌다 하기도 하고.. 😥 그래서 VS Code로 에디터를 바꿨는데 아주 만족하고 있다.😄 Atom 쓸 땐 뭔가 우왕좌왕 하게 됐는데 Vs Code는 편안하달까. 에디터를 열고 Jekyll 테마 내용물들이 들어 있는 깃허브아이디.github.io 이름의 로컬 폴더를 열어준다.



2. _posts 폴더를 생성한다.Permalink
깃허브아이디.github.io 이름의 로컬 폴더 위치에 _posts 폴더가 이미 있다면 냅두고 없다면 _posts 라는 이름의 폴더를 생성해 준다.

모든 포스트 파일은 이 _posts 내에 위치하여야 한다.


3. yyyy-mm-dd-title.md 형식의 파일을 만들어준다.Permalink
포스트 파일의 확장자는 md이어야 한다. yyyy-mm-dd 형식의 날짜와 함께 -포스트 제목을 붙여 준다. 포스트 제목은 영어로 쓸 것을 추천한다.

ex) 2023-06-16-my-first-post.md



4. 머릿말(Front-Matter)을 상단에 작성해 주어야 한다.Permalink
이제 md 파일에 포스트를 작성해보자. 내용을 작성하기 전에 이 포스트의 정보를 머릿말로 적어주어야 한다.

---
title:  "[Jekyll] 블로그 포스팅하는 방법"
excerpt: "md 파일에 마크다운 문법으로 작성하여 Github 원격 저장소에 업로드 해보자. 에디터는 Visual Studio code 사용! 로컬 서버에서 확인도 해보자. "

categories:
  - Papers
tags:
  - [Blog, jekyll, Github, Git]

toc: true
toc_sticky: true
 
date: 2023-06-16
last_modified_at: 2023-06-16


`원래 힘으로 돌아가려는 힘 벡터` = \\(\vec{f_{ij,spring}} = -k(\vert\vec{x_i} - \vec{x_j}\vert - l_0){\vec{x_i} - \vec{x_j}\over{\vert\vec{x_i} - \vec{x_j}\vert}} \\)

`원래 힘으로 돌아가려는 힘 벡터` = \\[\vec{f_{ij,spring}} = -k(\vert\vec{x_i} - \vec{x_j}\vert - l_0){\vec{x_i} - \vec{x_j}\over{\vert\vec{x_i} - \vec{x_j}\vert}} \\]


<iframe src="../paper/dimension_reduction/sne.pdf" width="100%" height="800px">
  <p>Unable to display PDF. Click <a href="../paper/dimension_reduction/sne.pdf">here</a> to download it.</p>
</iframe>

byebye
---
# bye
<a href="../paper/dimension_reduction/sne.pdf" class="image fit"><img src="images/marr_pic.jpg" alt=""></a>
    
---
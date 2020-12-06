# from FORK to PR

반갑습니다람쥐!  
<br>
![다람쥐 멋있는 자세](./images/다람쥐%20멋있는%20자세.JPG)  
<br>
깃허브 포크 기능과 풀 리퀘스트로 어떻게 백엔드팀 세미나 레포에 기여할 수 있는지 차근 차근 알아봅시다!  
( 첨부된 사진의 레포 주소와 실제 레포 주소는 다릅니다. )  

## 1. Fork 버튼 누르기
![Fork 버튼 누르기](./images/1.%20ForkButton.PNG)

[백엔드팀 Seminar 레포](https://github.com/mash-up-kr/Backend_9th_Seminar)에서 제목 오른쪽에 있는 `Fork` 버튼을 클릭합니다.  
성공적으로 Fork 가 되었다면, `mash-up-kr` 소유가 아닌 자신의 소유로 클론됩니다.  
<br>
![내 소유로 클론](./images/2.%20MyRepo.PNG)  
내 소유의 Seminar 레포를 Local 공간으로 클론합니다.  
본 설명에서는 [Sourcetree 어플리케이션](https://www.sourcetreeapp.com/) 기준으로 설명하겠습니다.  
<br>
![계정 추가](./images/6.%20SourceTree%20Account.PNG)
![계정 추가2](./images/7.%20SourceTree%20Account%20Add.PNG)
SourceTree 프로그램을 실행해 계정을 추가해줍니다.  
<br>
![.GIT](./images/8.%20Github%20clone%20https.PNG)
![clone](./images/9.%20SourceTree%20Clone.PNG)
본인 소유의 Seminar 레포의 .git 파일 주소를 가져와 로컬로 클론합니다.  
<br>
![plugin](./images/10.%20VSCode%20Markdown.PNG))

마크다운 에디터로 Visual Studio Code 를 사용합니다.  
좌측 다섯 번째 탭인 플러그인 탭에서 Markdown 을 검색하여 `Markdown All In One` 플러그인을 설치합니다.  
프로그램 재시작 후 마크다운 확장 파일을 열고 `Ctrl + Shift + P` 또는 `Command + Shift + P` 버튼으로 `Markdown: Open Locked Preview to the Side` 항목을 선택하고 `Enter` 키를 누릅니다.  
우측에 실시간으로 마크다운을 보여주는 창이 생깁니다.  
<br>
마크다운 문법은 [Github Markdown 가이드](https://guides.github.com/features/mastering-markdown/)를 참고해주시기 바랍니다.

## 2. 파일 생성하기
![파일 생성](./images/3.%20MyFolder.PNG)

적절한 폴더에 마크다운 파일을 생성합니다.  
이미지 파일을 삽입해야 하는 경우, `images/` 폴더를 만들어 그 안에 저장합니다.  
그 다음 필요한 경우, `README.md` 파일을 수정합니다.

![README파일 수정](./images/4.%20README.PNG)

## 3. 로컬 저장소에 '커밋' 하기

커밋 명명법은 `README.md` 파일에 있습니다.  
명명법에 맞춰 로컬 레포에 커밋을 합니다.  
<br>
![Staging](./images/5.%20SourceTree1.PNG)
먼저 Sourcetree 프로그램에서 언스테이징 파일을 스테이징 파일로 옮깁니다.  
언스테이징 파일은 커밋 메시지와 함께 커밋이 되지 않는 파일을 의미합니다.  
스테이징 파일로 이동해야 커밋 메시지가 담긴 커밋을 생성할 수 있습니다.  
<br>
![Commit](./images/11.%20Commit.PNG)
커밋 메시지와 함께 커밋 버튼을 누릅니다.  
`즉시 푸시` 항목을 체크하면, 커밋과 동시에 원격 저장소로 푸시가 바로 됩니다.  
커밋을 수정해야 할 때, 원격 저장소의 커밋들을 수정하려면 복잡하므로 로컬 커밋을 확정짓고 푸시하는 것이 좋습니다.  
[커밋 수정 문서 링크](https://github.com/HomoEfficio/dev-tips/blob/master/Git%20%EA%B3%BC%EA%B1%B0%EC%9D%98%20%ED%8A%B9%EC%A0%95%20%EC%BB%A4%EB%B0%8B%20%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0.md)  

## 4. 원격 저장소로 '푸시' 하기

![푸시버튼](./images/12.%20Push%20Button.PNG)  
푸시는 로컬 저장소에 기록된 커밋들을 원격 저장소로 보냅니다.  
SourceTree 의 상단 버튼 중 Push 버튼을 누릅니다.  
창이 뜨고 그대로 Push 버튼을 누릅니다.  
<br>
![Before](./images/13.%20Before%20Push.PNG)
푸시 전 커밋 저장소 모습입니다.  
<br>
![After](./images/14.%20After%20Push.PNG)
푸시 후 커밋 저장소 모습입니다.

## 5. 백엔드팀 레포로 Pull Request 보내기

Pull Request 기능은 GitHub 사이트에서 할 수 있습니다.  
Fork한 내 소유 레포의 변경 사항을 백엔드팀 레포로 전달하고 싶을 때, 백엔드팀 레포에 내 커밋 기록들을 받아들이라고 요청할 수 있습니다.

![PR](./images/15.%20PR.PNG)

PR 페이지는 내 소유의 레포에서 `Pull requests` 탭을 클릭하면 나옵니다.  
`New pull request` 버튼으로 새로운 PR 을 만듭니다.

![PR Success](./images/16.%20PR%20Success.PNG)

백엔드팀 레포로 내 변경 사항들을 넣을 수 있을 때, `Able to merge.` 메시지가 나타납니다.  
이 때, `Create pull request` 버튼을 누르면 백엔드팀 레포의 `Pull requests` 페이지로 이동됩니다.

![History](./images/17.%20History.PNG)

그 아래에는 어떤 문서, 어떤 코드가 변경되었는지 확인이 가능합니다.

![CreatePR Page](./images/18.%20Goto%20CreatePR.PNG)  

백엔드팀 레포의 `Open a pull request` 페이지입니다.  
PR 템플릿 내용이 미리 작성되어 있습니다.

![Write PR](./images/19.%20Write%20PR.PNG)

적절한 제목을 작성하고 PR 템플릿에 맞춰 내용을 작성합니다.

![PR Preview](./images/20.%20PR%20Preview.PNG)

Preview 탭에서 작성한 마크다운이 어떻게 나오는지 확인이 가능합니다.  
`Create pull request` 버튼을 눌러 백엔드팀 레포에 요청합니다.

## 6. Upstream 의 변경사항 가져오기

![Cannot AutoMerge](./images/21.%20Cannot%20AutoMerge.PNG)

위와 같이 `Can't automatically merge.` 문구가 떠도 Pull Request 를 신청할 수 있습니다.  
위 문구는 백엔드팀 레포와 포크한 레포의 커밋 기록이 두 갈래로 나뉘어져 있기 때문입니다.  
이를 해결하기 위해 백엔드팀 레포로부터 변경 사항들을 가져와야 합니다.  
이 때, 백엔드팀 레포를 보통 `upstream` 이라고 합니다.  

![Upstream 1](./images/22.%20Upstream1.PNG)

Sourcetree 프로그램에서 `upstream` 원격 저장소를 생성합니다.  
`저장소 메뉴` -> `저장소 설정(Shift + Ctrl + ,)` 메뉴를 클릭합니다.  
원격 저장소 경로를 추가합니다.  

![Upstream 2](./images/23.%20Upstream2.PNG)

원격 이름을 upstream 으로 짓고, 백엔드팀 레포의 .git 주소를 입력합니다.  
계정을 설정한 후 확인 버튼을 눌러 새로운 원격 저장소를 생성합니다.

![Upstream menu](./images/24.%20Upstream%20menu.PNG)

좌측 원격 트리에 `upstream` 에 오른쪽 마우스 버튼을 눌러 `upstream 에서 가져오기` 버튼을 누릅니다.

![Merge menu](./images/25.%20Merge%20menu.PNG)

브랜치 목록에서 `upstream/master` 브랜치로부터 가져온 커밋 항목을 오른쪽 마우스 버튼을 눌러 병합 메뉴를 누릅니다.  

![Conflict](./images/26.%20Conflict.PNG)

로컬 파일과 Upstream 파일이 일치하지 않기에, 충돌이 발생합니다.  
파일 내용을 보면

```
<<<<<<< HEAD
Upstream 브랜치 내용
========
병합할 브랜치(master) 내용
<<<<<<<
```

으로 변경되어있습니다.  
Upstream 브랜치 내용은 없어도 되니 삭제해줍니다.  
VS Code 에서 `Accepct Incoming Change` 버튼을 누르면 HEAD 가 아닌 두 번째 내용으로 변경됩니다.  

그 후 병합 이력을 커밋 명명법에 맞게 커밋하고 푸시합니다.

![Resolve conflict](./images/27.%20Resolve%20conflict.PNG)

Conflict 가 해결되었다면 `Able to merge` 로 다시 변경됩니다.

## 7. Pull Request Review

Pull Request 가 잘 보내졌다면 관리자의 리뷰가 시작됩니다.  
리뷰는 다음과 같이 진행됩니다.  

![PR Review1](./images/28.%20PR%20Review1.PNG)

리뷰어가 커밋 변경 기록들을 보고 코드에 댓글을 답니다.  
그리고 총 리뷰 글을 작성합니다.  

![PR Review2](./images/29.%20PR%20Review2.PNG)

모든 댓글이 해결이 된다면, 무사히 백엔드팀 레포 커밋에 합류하게 됩니다. :)

## 8. 마무리

긴 문서를 읽어주셔서 감사합니다.  
막히는 부분이 있다면 편하게 동아리원들에게 질문해주시기 바랍니다.
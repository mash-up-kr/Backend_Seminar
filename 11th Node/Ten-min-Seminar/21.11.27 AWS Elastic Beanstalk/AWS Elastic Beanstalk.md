# AWS Elastic Beanstalk

발표 날짜: 2021년 11월 27일
발표 여부: Yes
발표자: 김건회

### Elastic Beanstalk이란?

- 어플리케이션 설정, 생성, 배포, 관리를 빠르고 간단하게 지원해주는 개발자 풀 코스 서비스
- 자동화란 이런 게 아닌가 싶다
- 사용자가 빈스톡에서 몇몇 파라미터를 정의하고 코드를 올리면 AWS는 어플리케이션 실행에 필요한 인프라를 구성, 실행, 유지한다
- 빈스톡에 포함된 것
    - EC2 인스턴스 로드 밸런싱
    - Auto Scaling
    - RDS
    - 등등이 포함되어있음
    - 빈스톡을 사용하지 않으면 사용자가 직접 구성해야하는 것들
    - JAVA, Node.js, Python, Docker등등 여러 언어와 플랫폼 호환
    - 인프라 비용 외 추가 비용 없음

### 구축 과정

1.  생성 

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled.png)

일반적인 서버를 구축할 때는 웹 서버 환경 선택

배치나 스케줄 작업을 위해서는 작업자 환경을 선택(SQS Queue 사용)

---

**SQS Queue**

처리해야할 업무를 메세지라고 부르는 일종의 TODO리스트 형태로 저장

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%201.png)

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%202.png)

---

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%203.png)

환경 및 애플리케이션 이름 설정

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%204.png)

플랫폼 설정

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%205.png)

추가 옵션들 중 가장 흥미로웠던 건 **롤링 업데이트와 배포**

롤링 업데이트. 왠지 이름이 멋있다. ~~롤링썬더!~~

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%206.png)

---

**롤링 ~~썬더~~ 업데이트**

- 배포방식
    - 한 번에 모두
        - 모든 인스턴스에 동시에 새 버전 배포
        - 빠른 배포
        - 다운타임 생김
    - 변경 불가능
        - 기존 인스턴스 유지
        - Auto Scaling 배치 그룹 생성하여 배포
- 구성 업데이트 - 롤링 업데이트
    - 비활성(Rolling)
        - 기존 인스턴스 중 일부를 배치 단위로 선정하여 배포
        - 다운타임 없애기 위해 일부씩 업데이트
    - 변경 불가능
        - 

|  | 가용영역 | 로드밸런서 | Auto Scailing |  |
| --- | --- | --- | --- | --- |
| 단일 인스턴스(프리 티어 사용 가능) | 선택불가 | X | X |  |
| 단일 인스턴스(스팟 인스턴스 사용) | 선택불가 | X | X |  |
| 고가용성 | ~3개까지 | O | O |  |
| 고가용성(스팟 및 온디맨드 인스턴스 사용 | ~3개까지 | O | O |  |

---

**스팟 인스턴스란?**

- **스팟 인스턴스**는 **사용자 제시 가격(입찰가격)**을 **정해놓고** **저렴할 때 이용**할 수 있습니다.
    - **사용자가 제시한 가격**보다 **인스턴스 시장 가격**이 **높아지게 되면** **인스턴스가 종료**됩니다.
        - 시장 가격은 **인스턴스 패밀리, 인스턴스 크기, 가용 영역(AZ), 리전(Region)** 등에 따라 달라집니다.
        - 또한, **수요와 공급량**에 따라 가격이 달라집니다.
        - **종료 되는 시점을 알 수는 없습니다.**
    - 하지만 여유 자원에 대한 경매 방식으로 **온디멘드 대비 80~90% 저렴**합니다.
- 장점
    - **1~6시간 이내 짧은 워크로드**를 다루거나, **갑작스런 피크 타임**에 해당하는 컴퓨팅 리소스를 확보해야 할 때 좋다.
    - 즉, **단기적으로 수요가 많을 때** 유리하다.
    - 주로 Batch Job 등 용도로 사용하는 서버에 잘 어울린다.

---

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%207.png)

EB가 만들어지는 동안 IAM *및* Github 세팅을 해봅시다

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%208.png)

IAM 에 들어가서 사용자를 추가해준다

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%209.png)

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%2010.png)

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%2011.png)

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%2012.png)

- .github/workflows/deploy.yaml 파일
    
    ```yaml
    name: Deploy to EB
    # main branch에 push 되면 실행
    on:
      push:
        branches:
          - main
    
    jobs:
      InstallDependency:
        name: CI Pipeline
        runs-on: ubuntu-18.04
        strategy:
          # Node version은 12 버전을 이용한다.
          matrix:
            node-version: ["12.x"]
    
        steps:
          - uses: actions/checkout@v2
    
          # Initialize Node.js
          - name: Install Node.js ${{ matrix.node-version }}
            uses: actions/setup-node@v1
            with:
              node-version: ${{ matrix.node-version }}
    
          # Install project dependencies, test and build
          - name: Install dependencies
            run: npm install
    
      deploy:
        name: CD Pipeline
        runs-on: ubuntu-18.04
    
        strategy:
          matrix:
            node-version: ["12.x"]
        # 위의 InstallDependency가 실행되고 진행된다.
        needs: InstallDependency
        steps:
          - uses: actions/checkout@v2
          # env 파일을 이용할 일이 보통 많은데,
          # Github Secrets를 이용하여 env 파일을 만들고 추가한다.
          # 참고로 ElasticBeanstalk에 Node 관련을 배포할 때는,
          # 8081 포트를 열어줘야 한다!!
          - name: Create env file
            run: |
              touch .env 
              echo PORT=8081 >> .env 
              echo NODE_ENV=${{ secrets.NODE_ENV }} >> .env 
              cat .env
          # Initialize Node.js
          - name: Install Node.js ${{ matrix.node-version }}
            uses: actions/setup-node@v1
            with:
              node-version: ${{ matrix.node-version }}
    
          # Install project dependencies and build
          - name: Install dependencies
            run: npm install
    
          # Install AWS CLI 2
          - name: Install AWS CLI 2
            run: |
              curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
              unzip awscliv2.zip 
              which aws 
              sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
          # Configure AWS credentials
          - name: Configure AWS credentials
            uses: aws-actions/configure-aws-credentials@v1
            with:
              aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
              aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
              aws-region: ${{ secrets.AWS_REGION }}
    
          # Make ZIP file with source code # -x는 zip파일 생성 시에 해당 부분들을 제외한다.
          - name: Generate deployment package
            run: zip -r deploy.zip . -x '*.git*' './aws/*' awscliv2.zip
    
          # Deploy to Elastic Beanstalk
          # application_name과 environment_name을 꼭 확인하자!
          # 해당 부분은 꼭 같아야 한다!!
          - name: Deploy to EB
            uses: einaregilsson/beanstalk-deploy@v14
            with:
              aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
              aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
              application_name: aws-beanstalk-mash-up-test
              environment_name: Awsbeanstalkmashuptest-env
              region: ${{ secrets.AWS_REGION }}
              version_label: ${{github.SHA}}
              deployment_package: deploy.zip
    ```
    

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%2013.png)

직접 해보면서 느낀 건 배포가 정말 말도 안되게 편하다는 거였다.

EC2인스턴스를 만들고 들어가서 노드를 깔고 클론을 받고 어쩌고 저쩌고의 과정이 전혀 필요가 없다..

자동 로드밸런싱과 오토 스케일링까지 지원해주니 엥간한 경우에는 EB를 적극 사용해도 괜찮지 않나 라는 생각이 들었다.

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%2014.png)

처음 내가 ElasticBeanstalk를 알게 된 건 요우라는 개발자 분의 [이력서 페이지](https://resume.yowu.dev/)를 보면서였다.

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%2015.png)

이 분의 커리어패스를 닮고 싶다는 생각이 들어서 자주 들어가보는 편인데 이번 발표 때 무엇을 발표하면 좋을까 고민하다가 자주 들어본 것에 비해 사용해본 적은 없는 거 같아 소개해보게 되었다.

이 글을 쓰면서 발견한 건데 저기 위에 보면 Mashup API라고 해서 우리 동아리와의 관련성을 잠시나마 의심했지만 Mashup API를 검색해보고 엔프피의 망상이였다는 걸 깨달았따

![Untitled](AWS%20Elastic%20Beanstalk%20ab2c02c34b604a47ab762f0f51782e83/Untitled%2016.png)

[Qwiklabs](https://run.qwiklabs.com/)

이건 이번 발표와는 관계 없는 사이트인데 클라우드 서비스를 체험 및 실습해볼 수 있다고 한다(무료로)

Reference

[9분 59초 만에 Github Action + AWS Elastic Beanstalk로 TS 프로젝트 CI/CD 파이프라인 구축하기](https://bluayer.com/46)

[[AWS] AWS Elastic beanstalk 을 이용한 웹어플리케이션 구축 - 1](https://yonguri.tistory.com/35)
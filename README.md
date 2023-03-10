# week-1-2 (원티드 프리온보딩 프론트엔드)

## CI/CD 과정

1. AWS S3 bucket 생성
2. CI/CD  
   (1) test 코드 작성  
   (2) Github Actions에 AWS key 등록  
   (3) [파일명].yml파일 생성

   ```
   name: CI/CD

   on:
     push:
       branches:
       - master
     workflow_dispatch:

   jobs:
     cicd:
       runs-on: ubuntu-latest
       steps:
       - uses: actions/checkout@master
       - run: npm install dependency 설치
       - run: npm run test
       - run: echo SUCCESS
           - run: echo TEST_SUCCESS
           - run: npm run build
           - uses: jakejarvis/s3-sync-action@master
             with:
               args: --delete
             env:
               AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
               AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
               AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
               AWS_REGION: 'ap-northeast-2'
               SOURCE_DIR: 'build'
   ```

### 자동화 첫 단계

1. 터미널 명령어 사용

### aws build & 업로드 자동화에 필요한 과정

1. build 후
2. 서버에 업로드
3. 서빙

### aws build & 업로드 자동화 실습

1. 초기화

```
npx create-react-app .
npm run build
```

- npm run build 후에

```
npm run build
```

- build 파일을 서버에 업로드, 다시 서빙

```
npm install -g serve
serve -s build
```

- 만약 S3에 업로드 한다면, 드래그앤드롭으로 업로드 하자.
- 파일, 폴더 모두 한번에 업로드 가능  
  <br/>

1. AWS 로그인

```
//aws configure //처음인 경우
//aws configure default profile //profile 지정
aws configure --profile [profile name]
```

- 아이디, 비밀번호, ap-northeast-2(리전), 엔터

```
aws s3 ls s3://[버킷 이름] --profile [profile name]
```

![image](https://user-images.githubusercontent.com/86847564/221370756-63125f7b-25c4-4279-a312-af2f59c9b16f.png)

<br/>
2. 업로드

- 내 특정 폴더와 aws sync 맞추기

```
npm run build //빌드

cd [빌드 파일이 있는 경로] //경로 이동
aws s3 sync build/ s3://[버킷 이름] --profile [profile name] --delete

aws s3 sync build/ s3://s3-deploy-cicd-9th-test --profile wanted-session --delete //기존꺼 삭제하고 빌드 파일 업로드
```

- 출력 결과 없음

<br/>
3. package.json에 스크립트 입력. 
- 빌드 후 이전에 업로드한 파일/폴더 삭제, 자동으로 업로드

```
// package.json

"scripts": {
    "deploy": "npm run buid && aws s3 sync build/ s3://[버킷 이름] --profile [profile name] --delete
"
}
```

CDN: cloudfront, Content Delivery Network  
Route 53: 도메인 연결

<br/>

### CI/CD: 자동화

- CI: merge, test. '자주' branch 만들어서 test 하기
- CD: deployment.
  - Continuous Delivery: 개발 환경의 배포 자동화
  - Continuous Deployment: Production 환경까지 배포 자동화
- Github Actions=> 클라우드형 CI/CD 플랫폼  
  <br/>

### 1. Github Actions 구성요소

(1) Workflow: 파이프라인. YAML 형식 파일  
(2) Event: 레파지토리에서 발생하는 push, pull request open, issue open 활동 (예) 특정 Event 발생 시 해당 CI/CD 파이프라인 구동  
(3) Jobs: step 그룹. step을 순차적 실행  
(3-1) step: 터미널 명령어 한 줄  
(4) Actions: GitHub workflow에서 자주 사용되는 기능들을 모아 놓은 커스텀 애플리케이션. 라이브러리 같은 것. Market place에서 찾을 수 있다.  
(5) Runner: 컴퓨터

### 2. CI/CD로 구축할 목표

(1) master branch에 push, pr merge 이벤트 발생시 workflow 실행  
(2) dependencies 설치  
(3) build script 실행  
(4) aws 접속, s3 bucket에 build 결과 업로드

### CI 과정

> (1) 마스터 브랜치에 push, pr merge 이벤트 발생시 workflow 실행

- test 코드 작성
- npm run test
- master branch에 push 했을 때 통과할지 여부 검사

- github repository의 Actions 버튼 클릭

1. <span style='background-color:#dcffe4'>github repo/.github/workflows/[파일명].yml</span> 이 있다면 실행이 가능하다.  
   따라서, 내 로컬 repo 안에 .github폴더 안에 workflows 폴더를 만들고, 다음을 입력한다.

```
name: CI/CD //이 workflow의 이름

on: //이 workflow가 언제 실행될 것인지
  push: //push 됐을 때
    branches:  //어떤 branch?
    - master //master라는 branch. master에 push 됐을 때.
             //여러 브랜치 가능.

jobs: //무슨 동작?
  cicd: //cicd라는 job
    runs-on: ubuntu-latest //ubuntu 최신버전 빌려주세요
    steps: //뭘 할까?
    - uses: actions/checkout@master //master branch로 checkout
    - run: npm install //npm install. dependency 설치
    - run: npm run test //test 명령어 실행
    - run: echo SUCCESS //터미널에 SUCCESS 출력

```

- commit & push 진행

- 위의 uses: actions/checkout@master //master branch로 checkout 에서 action은 어디에서 찾을 수 있을까 => <span style='background-color:#dcffe4'>Github의 Market place</span>

![image](https://user-images.githubusercontent.com/86847564/221394896-a751c699-d582-4d4b-bfc9-0b5c812eeed6.png)

![image](https://user-images.githubusercontent.com/86847564/221394933-c041c6a0-713f-4f06-9f85-489cf7065522.png)

- 커스텀 사용이 가능

- 실행 결과
  ![image](https://user-images.githubusercontent.com/86847564/221395342-01040d44-164f-4bc8-9b58-d33746ee08e8.png)

2. 만약 push할때만이 아니라, 깃헙에서 버튼 눌러서 아무때나 해보고 싶다면?=> [파일명].yml파일에 <span style='background-color:#dcffe4'>workflow_dispatch:</span> 추가

```
name: CI/CD //이 workflow의 이름

on: //이 workflow가 언제 실행될 것인지
  push: //push 됐을 때
    branches:  //어떤 branch?
    - master //master라는 branch. master에 push 됐을 때.
             //여러 브랜치 가능.
  workflow_dispatch:

jobs: //무슨 동작?
  cicd: //cicd라는 job
    runs-on: ubuntu-latest //ubuntu 최신버전 빌려주세요
    steps: //뭘 할까?
    - uses: actions/checkout@master //master branch로 checkout
    - run: npm install //npm install. dependency 설치
    - run: npm run test //test 명령어 실행
    - run: echo SUCCESS //터미널에 SUCCESS 출력
```

![image](https://user-images.githubusercontent.com/86847564/221396309-2f9a3ae0-7460-46b6-95b0-a0f747efb0be.png)

- Actions의 CI/CD에 들어가면 우측에 'Run workflow'버튼

![image](https://user-images.githubusercontent.com/86847564/221396345-4ed8c8b0-61e3-4351-972a-9f75d8273621.png)

### CD: build, S3에 업로드

(1) aws 로그인  
(2) cli 설치  
(3) s3 에 설치  
=> 오 복잡해!: 누군가는 만들어 놓았음. Market place!  
<span style='background-color:#fff5b1'>s3</span> 입력 후, <span style='background-color:#fff5b1'>Most installed/starred</span> 정렬 버튼 클릭  
![image](https://user-images.githubusercontent.com/86847564/221396851-ced636d4-620d-4b4a-998f-cdd52a2e3ad9.png)

```
name: CI/CD //이 workflow의 이름

on: //이 workflow가 언제 실행될 것인지
  push: //push 됐을 때
    branches:  //어떤 branch?
    - master //master라는 branch. master에 push 됐을 때.
             //여러 브랜치 가능.
  workflow_dispatch:

jobs: //무슨 동작?
  cicd: //cicd라는 job
    runs-on: ubuntu-latest //ubuntu 최신버전 빌려주세요
    steps: //뭘 할까?
    - uses: actions/checkout@master //master branch로 checkout
    - run: npm install //npm install. dependency 설치
    - run: npm run test //test 명령어 실행
    - run: echo TEST_SUCCESS //터미널에 TEST_SUCCESS 출력
    - run: npm run build
    - uses: jakejarvis/s3-sync-action@master // 이 action 사용. action 이름
      with:
        args: --delete //argument는 delete만. sync할 때 옵션
      env:
        AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }} //어떤 bucket
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }} //인증. 환경변수
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }} //인증. 환경변수
        AWS_REGION: 'ap-northeast-2'
        SOURCE_DIR: 'build' // build를 업로드
```

- github Actions>Secrets and variables> New Repository secret버튼 클릭

![image](https://user-images.githubusercontent.com/86847564/221397126-4b4a2c27-59d0-466e-b2c7-2adaa6ee8b2e.png)

- Actions secrets/New secret 입력(추후에는 수정만 가능. 볼 순 없다.)

  - Name: AWS_S3_BUCKET <span style='background-color:#dcffe4'>AWS_S3_BUCKET</span>//yml 파일 확인
  - Secret: bucket 이름

- 다시, New Repository secret버튼 클릭

- Actions secrets/New secret

  - Name: <span style='background-color:#dcffe4'>AWS_ACCESS_KEY_ID</span>//yml 파일 확인
  - Secret: AWS_ACCESS_KEY_ID에 해당하는 값

- Actions secrets/New secret
  - Name: <span style='background-color:#dcffe4'>AWS_SECRET_ACCESS_KEY</span>//yml 파일 확인
  - Secret: AWS_SECRET_ACCESS_KEY에 해당하는 값

## Windows에서 `virtualenv`로 가상환경 세팅하기

#### 1. `virtualenv` 설치
```
$ pip install virtualenv
```

#### 2. python.org에서 원하는 파이썬 버전 설치 (파이썬 설치경로 잘 기억해두기, 설치 시 `Add path` 꼭 체크하기)
> 나는 python 3.6.8 버전 다운로드
>
> 3.6.8의 설치경로는 C:/Users/USER/AppData/Local/Programs/Python/Python36/python.exe

#### 3. virtualenv로 가상환경을 만들면 폴더가 하나 생성되기 때문에, git에 추적?되지 않기 위해 가상환경을 저장할 폴더를 새로 생성하기
```
$ cd workspace
$ mkdir myvenv
```
**myvenv 폴더로 이동**
```
$ cd myvenv
```
**virtualenv로 python 3.6 버전 가상환경 만들기**
```
$ virtualenv [가상환경이름] -p C:/Users/USER/AppData/Local/Programs/Python/Python36/python.exe

-- Python 버전명을 명시하여 만들기
$ virtualenv [가상환경이름] --python=python3.8.6
```


#### 4. 가상환경을 실행할 프로젝트 폴더로 이동 후 virtualenv activate
```
$ cd ..
$ cd [프로젝트이름]
$ source ../myvenv/[가상환경이름]/scripts/activate
```

### 5. virtualenv 종료
```
$ deactivate
```

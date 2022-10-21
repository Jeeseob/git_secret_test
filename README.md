# git_secret_test


### 1. git secret install


```
$ apt install git-secret
```

###  2. gpg install
```
$ apt install gpg
```

### 3. gpg key 생성
```
$ gpg --full-generate-key
gpg (GnuPG) 2.2.4; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)

Your selection?  1 
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)  0 
Key does not expire at all
Is this correct? (y/N)  y 

GnuPG needs to construct a user ID to identify your key.

Real name:  [My Name]
Email address:  [My Email]
Comment:  [My Comment] 
You selected this USER-ID:
    "[My Name] ([My Comment]) <[My Email]>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?  O 
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

```




### 4. git-secret 암호화
```
$ git secret init
# git secret tell [My Email]

$ git secret add [secret file]
$ git secret hide

## [secret file].secret 파일이 생성되었는지 확인
## 이후 git add / commit / push
```


### 5. private key 생성
```
$ gpg --list-secret-keys
$ gpg -a --export-secret-keys $keyid > gpg-secret.key
$ gpg --export-ownertrust > gpg-ownertrust.txt

## 해당 파일은 github으로 보내지지 않도록 주의
```



### 6. Jenkins 적용

#### 6-1. Jenkins에 Credential로 등록

Secret file로 추가
File: gpg-secret.key
ID: gpg-secret    
![image](https://user-images.githubusercontent.com/81093419/197101273-db3bfdec-f920-4675-8ef2-30d641ad2f04.png)
   
     
Secret file로 추가
File: gpg-ownertrust.txt  
ID: gpg-ownertrust    
![image](https://user-images.githubusercontent.com/81093419/197101314-abe36740-1164-4092-93fb-efb663a72d62.png)

Kind: Secret text   
Text: <Passphrase used to generate GPG key>   
ID: gpg-passphrase   
![image](https://user-images.githubusercontent.com/81093419/197101323-1da3f632-c853-4784-b9cc-8128e72631a3.png)


#### 6-2. Jenkinsfile에 추가

```
## Jenkinsfile의 enviroment에 credentials 추가
pipeline {
  agent any
  environment {
    gpg_secret = credentials("gpg_secret")
    gpg_trust = credentials("gpg_trust")
    gpg_passphrase = credentials("gpg-passphrase")
  }
 stages {
    ...
 }
}
  
  
## Jenkinsfile에 build 이전에 stage로 추가

stage('get git secret'){
  agent any
  steps {
      sh ("gpg --batch --import $gpg_secret")
      sh ("gpg --import-ownertrust $gpg-ownertrust")
      sh ("git secret reveal -p '$gpg_passphrase'")
  }
}
```


만약 처리가 안된다면, Jenkins server에 Git secret과 gpg를 설치한 후 진행해 볼 것.

참고 :    
https://www.baeldung.com/ops/jenkins-inject-git-secrets   
https://jhyutno.tistory.com/entry/git-secret

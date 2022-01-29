CMF 개발 절차

 ELG로부터 요구사항을 받아서 최종 PKG release 까지의 과정을 간략히 정리해서 공유해 드립니다. 

  

JIRA 통해 업무 접수 시 : Select for Development 선택 

업무 Start 시 : "작업흐름"에서 In Progress 선택 

  

1. coding 

말 그대로 요구사항에 맞게 coding 하는 과정입니다. 

  

2. build 

컴파일 및 packaging 과정이라고 생각하시면 됩니다. 

이미 알고 계시겠지만 

source build/local/clasp.env (RHEL7 기준으로 build 환경 변경) 

make -j20 

make image (./target/local/image 디렉터리 아래 mediation.tgz 파일 생성) 

와 같이 진행하시면 됩니다. 

------------------------------------------------------------------------------------------------
[GIT 사용 소스 개발 절차]

1. 개발 서버 (10.180.92.250) 에 Z-ID로 로그인 
- git config 설정 
  git config --global user.name "Kimoon Han" 
  git config --global user.email sungkyu.han@ericsson.com 
  git config --global core.editor vim 
  git config --global color.status auto 
  git config --global color.branch auto 
  git config --global color.interactive auto 
  git config --global color.diff auto 
  git config --global alias.dog "log --all --decorate --oneline --graph" 

2. ssh-keygen 으로 key 생성하여 pub key를 gerrit site에 등록  (FireFox 브라우저 사용 필요)

3. git clone {URL}     ; hook 걸려있는 소스로 clone 해야 commit 가능함 
   --> URL은 gerrit확인: Project -> Filter(CMF_Charging) --> Clone with commit-msg hook (copy to clipboard 클릭복사)
        git clone https://znhakim@gerrit.ericsson.se/a/elgcloudca/CMF_Charging && (cd CMF_Charging && mkdir -p .git/hooks && curl -Lo `git rev-parse --git-dir`/hooks/commit-msg https://znhakim@gerrit.ericsson.se/tools/hooks/commit-msg; chmod +x `git rev-parse --git-dir`/hooks/commit-msg)
        git clone https://znhakim@gerrit.ericsson.se/a/elgcloudca/CMF_Charging && (cd CMF_Charging && mkdir -p .git/hooks && curl -Lo `git rev-parse --git-dir`/hooks/commit-msg https://znhakim@gerrit.ericsson.se/tools/hooks/commit-msg; chmod +x `git rev-parse --git-dir`/hooks/commit-msg)

4. git branch -r    ; LGU/SKT SA/NSA PKG별 Branch 확인
- origin/LR2.1.0    ; LGU+ SA 신형 PKG (현재 DVT 수검중인 PKG)
- origin/LR1.0.0    ; LGU+ NSA 
- origin/R1.4.0     ; SKT NSA
- origin/SR1.0.0    ; SKT SA

- origin/LR2.1.1 ;LGU+ SA 성능시험PKG

5. git branch -b LGU_SA origin/LR2.1.1    ; branch 이름 만들기 (예)
6. git checkout LGU_SA                       ; LGU SA branch로 이동
    ==> 5,6번 통합
   git checkout -b LGU_SA origin/LR2.1.0  --> LGU_SA
   git checkout -b LGU_NSA origin/LR1.2.0 --> LGU_NSA

git checkout -b LGUSA_PERFORMANCE origin/LR2.1.1  --> LGU_SA 성능시험용

7. 소스 수정 및 개발 (build 및 SIT 완료)
8.  git add [--all]           ; 소스코드 반영
  - git add FileName
    or
  - git add .

9. git commit -m "{JIRA 번호(예: ELG-nnnn)} 영어로 comment"
10. git pull --rebase     ;    (Password 필요) 소스 conflict 여부 확인
11. git push origin HEAD:refs/for/LR2.1.0         ; gerrit을 통한 소스 등록
12. Gerrit 서버 접속하여 Topic (#ELG-nnnn)과 Reviewers (Hyunggon Cho) 등록
   - https://gerrit.ericsson.se/#/admin/projects/
13. JIRA에 보완내역서(시험 결과 등) 등록 및 댓글에 "SIT & PUSH 완료" 올리고 담당자를 검증 담당자로 지정
------------------------------------------------------------------------------------------------


3. 테스트 및 보완내역서 작성 

2번 과정에서 생성된 package 파일(~/target/local/image/mediation.tgz)을 테스트 서버(Ctrix 통해 접속)에 업로드 하시고, 

테스트를 진행하시면 됩니다. 

테스트가 정상적으로 완료되면, 첨부파일과 같이 보완내역서를 작성해 주셔야 합니다(사이즈가 커서 압축해서 첨부했습니다). 

문서 제목이나 내용을 보면 아시겠지만, 해당 기능 개발을 위해 보완된 내역 및 테스트 방법 등을 작성하시면 됩니다. 

자세한 작성 방법에 대해서는 나중에 따로 문의하시면 알려드리겠습니다. 

  

일반적으로 CMF-A를 active로 사용하고 있기 때문에 CMF-A 장비에 package를 설치하시면 되는데요. 

혹시라도 CMF-B가 active로 사용될 수도 있으니  

적용 전에 “pcs status --full” 명령으로 어느 장비가 active인지 확인을 하시는 것이 좋을 것 같습니다. 

  

/opt/mediation/etc/init.d/medi-si stop svc (system, svc, all)명령으로 application을 중지하시고 

/opt/mediation/etc/init.d/medi-si status 상태확인 

  

cd /opt/mediation 디렉토리에서 package에 대한 압축을 해제하시면 됩니다 

tar xvfz mediation.tgz 

  

/opt/mediation/etc/init.d/medi-si start svc  로 재실행  

  

4. source commit & push 

개발이 완료된 소스를 git 서버에 등록하는 과정이라고 생각하시면 됩니다. 

git add --all apps/* //*/ 

git commit -m “내용” 

git pull --rebase // conflict 예방 

git push origin HEAD:refs/for/branch명 git push origin HEAD:refs/for/SR1.0.0 

? git commit 하실 때 comment 부분은 “[JIRA-NO] 변경한 내역에 대한 간단한 comment“와 같은데요. 

? 예를 들어, 현재 진행중이신 건에 대해서는 “[ELGCO-131] Add monitor group related parameters to command”와 같이 적당한 문구로 작성을 하시면 됩니다. 

? push 하실 때 branch 명은 SKT NSA의 경우 “R1.4.0”, SA의 경우 “SR1.0.0”으로 하시면 됩니다. 

? 참고로 git pull --rebase를 수행하는 이유는 혹시 이용구님께서 개발하시는 중에 다른 분이 push 한 소스가 있을 경우를 위해 최종 버전으로 동기화 해주는 과정입니다. 

? 만약 위 과정을 생략했을 경우에는 원격지 서버와 코드 내용이 달라서 conflict이 발생할 수도 있습니다. 

  

5. Gerrit에 reviewer 등록 

push가 완료된 이후 Gerrit에 접속하시면 “My > Changes” 메뉴를 선택하시면, “Outgoing reviews” 부분에 push 하신 정보가 보일텐데요. 

(참고로 Incoming reviews에는 다른 개발자 분이 push 올린 내용이 보입니다.) 

push 하신 항목을 선택하시면, 화면 중간 즈음에 “Reply…” 버튼이 보이고, 아래에 Reviewers, Topic 등이 보일겁니다. 

Topic에는 # + JIRA의 이슈 번호(#ELGCO-131)를 설정하시고, 

Reviewers에는 “Hansul Lee”, “Hyunggon Cho”, “Sungkyu Han”, “Gunbo Pak”을 추가하시면 됩니다. 

  

6. Jira에 개발 완료 처리 

마지막으로 Jira에 접속하셔서 완료 처리를 해 주셔야 하는데요. 

댓글에 “SIT&PUSH 완료했습니다” 등과 같이 comment를 작성해 주시고, 

작성하신 보완내역서를 업로드 해주세요. 

그리고 상태를 “In Progress”로 변경하시고, 담당자를 검증 담당자로 변경하시면 됩니다. 

참고로 검증 담당자는 SKT NSA의 경우 “Heechan Park”, SA의 경우 “Seungjin Ha”입니다. 

  

  

ELGCO-152 [CMF] msisdn 값이 0(NULL) 인 경우 과금 미생성 및 FLT 출력 기능 개발 요청 건 

"상태" 진행 중으로 변경하고  "담당자"를 이한설 님으로 변경 했습니다. 

JIRA 통해 업무 접수 시 : Select for Development 선택 

업무 Start 시 : "작업흐름"에서 In Progress 선택 


-------------------------------------------
# Openshift 주요 명령어

### 1. 프로젝트의 POD 정보 가져오기
```
# oc get pod -n [프로젝트명]
```
### 2. POD 원격 접속
```
# oc rsh [Running 중인 POD 명]
```
### 3. POD 내 JBOSS 재시작
```
# cd /opt/eap/bin
#./jboss-cli.sh --connect
# shutdown 
```

### 4. POD 쓰레드 덤프
```
# oc rsh [Running 중인 POD 명]
# ps -ef | grep java
# kill -3 [PID]
```
### 5. 비정상 POD 삭제
```
# oc get pod –n [프로젝트 명]
# oc delete pod [POD명] –n [프로젝트 명]
```

### 6. Running 중인 POD의 Node 확인
```
oc get pod --all-namespaces -o wide | grep Running
```

### 7. Application SSL 적용 방법
```
# oc get route -n [프로젝트명]
# oc edit route [라우터명]
 - spec:

host: jpapi.com
  tls:
    insecureEdgeTerminationPolicy: Allow
    termination: edge
  to:
    kind: Service
          name: jpapi
```

- 웹 콘솔에서 특정 route에 SSL 적용
- Applications > Routes > Route명 > Actions >Edit

### 8. POD 내의 파일 및 폴더를 로컬로 가져오기
```
# oc get pod –n [프로젝트 명]
# oc rsync [Running 중인 POD명]:[POD내 경로] [로컬 폴더]

ex) oc rsync wizard-173-4swgn:/opt/eap/standalone/configuration /home/tmp1234

# oc rsync [Running 중인 POD명]:[POD내 경로] [로컬 폴더]

ex) oc rsync wizard-173-4swgn:/opt/eap/standalone/configuration/standalone-openshift.xml /home/tmp1234
```

### 9. 로컬 폴더를 POD로 보내기
```
# oc get pod –n [프로젝트 명]
# oc rsync [로컬폴더] [Running 중인 POD명]:[POD내 경로]

ex) oc rsync /home/tmp1234 wizard-173-4swgn:/home/jboss/test/
```

### 10. POD 내 명령어 실행
```
# oc get pod

# oc exec [Running 중인 POD명] -- [명령어]

ex) oc exec wizard-173-4swgn -- date

ex) oc exec wizard-173-4swgn -- ls -al /
```

### 11. POD 내 JBoss PID 찾기 exec 방법
```
# oc get pod –n [프로젝트명]

# oc exec [Running 중인 POD명] -- ps –ef | grep java

ex) oc exec wizard-173-4swgn -- ps –ef | grep java

# oc exec wizard-173-4swgn -- kill -3 [PID]

ex) oc exec wizard-173-4swgn -- kill –3 211
```

### 12. POD 내 JBoss Heap Dump 뜨기
```
# oc get pod –n [프로젝트명]

# oc exec [Running 중인 POD명] -- ps –ef | grep java

ex) oc exec wizard-173-4swgn -- ps –ef | grep java

# oc exec [Running 중인 POD명] -- jmap -dump:live,format=b,file=[Dump 경로 및 파일명] [PID]

ex) oc exec wizard-173-4swgn --jmap -dump:live,format=b,file=/WORK/DUMP/test.dump 211
```

### 13. POD 증가 및 감소
```
# oc get dc –n [프로젝트명]

ex) oc get dc –n mojdev

# oc scale --replicas=[POD수] dc [배포설정명]

ex) oc scale --replicas=2 dc wizard
```

### 14. 프로젝트 빌드 관리 명령어
```
# oc get bc –n [프로젝트명]

# oc start-build [빌드명]

ex) oc start-build wizard

# oc cancel-build [Starded된 빌드명]

ex) oc cancel-build wizard-6

# oc get builds -n [프로젝트명]

ex) oc get builds -n mojdev
```

### 15. Openshift 배포/빌드/이미지 정리
```
# oc login –u [계정명] –p [패스워드]

# oc adm prune deployments --orphans --keep-complete=5 --keep-failed=1 --keep-younger-than=60m

# oc adm prune builds --orphans --keep-complete=5 --keep-failed=1 --keep-younger-than=60m

# oc adm prune images --keep-tag-revisions=5 --keep-younger-than=60m

 
- 배포/빌드/이미지 정리 실행

# oc adm prune deployments --orphans --keep-complete=5 --keep-failed=1 --keep-younger-than=60m --confirm

# oc adm prune builds --orphans --keep-complete=5 --keep-failed=1 --keep-younger-than=60m --confirm

# oc adm prune images --keep-tag-revisions=5 --keep-younger-than=60m --confirm
```

### 16. Docker 컨테이너 및 이미지 삭제
```
# docker rm $(docker ps –qa)
# docker rmi $(docker images | grep 172 | awk '{print $3}')
```

### 17. POD AutoScale 설정
```
# oc get hpa -n [프로젝트명]

ex) oc get hpa -n mojdev

# oc edit hpa [hpa명]

ex) oc edit hpa wizard

apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend
spec:
  scaleTargetRef:
    kind: DeploymentConfig
    name: frontend
    apiVersion: apps/v1
    subresource: scale
  minReplicas: 1
  maxReplicas: 4
  cpuUtilization:
    targetCPUUtilizationPercentage: 80
```

### 18. 프로젝트 Quota 설정
```
# oc get quota -n [프로젝트명]

ex) oc get quota -n dev

# oc get quota -n [프로젝트명] [Quota명] -o json

ex) oc get quota -n dev test-quota -o json

# oc edit quota -n [프로젝트명] [Quota명]

ex) oc edit quota -n dev test-quota

{
    "apiVersion": "v1",
    "kind": "ResourceQuota",
    "metadata": {
        "name": "test-quota"
    },
    "spec": {
        "hard": {
            "requests.memory": "4Gi",
            "requests.cpu": "2",
            "limits.memory": "4Gi",
            "limits.cpu": "4",
            "pods": "2"
        }
    }
}
```

### 19. 프로젝트의 POD 및 Container Limits 설정
```
# oc get limits -n [프로젝트명]

ex) oc get limits -n dev

# oc get limits -n [프로젝트명] [Limits명] -o json

ex) oc get limits -n dev test-limits -o json

{
    "kind": "LimitRange",
    "apiVersion": "v1",
    "metadata": {
        "name": "test-limits",
        "creationTimestamp": null
    },
    "spec": {
        "limits": [
            {
                "type": "Pod",
                "max": {
                    "cpu": "1000m",
                    "memory": "2Gi"
                },
                "min": {
                    "cpu": "1000m",
                    "memory": "2Gi"
                }
            },
            {
                "type": "Container",
                "max": {
                    "cpu": "1000m",
                    "memory": "2Gi"
                },
                "min": {
                    "cpu": "1000m",
                    "memory": "2Gi"
                },
                "default": {
                    "cpu": "1000m",
                    "memory": "2Gi"
                }
            }
        ]
    }
}
```

### 20. POD Terminating 상태 풀기
```
프로젝트 내 Terminating 중인 POD에 대해서 oc delete pod로 죽지 않는 경우 kill 하는 방법으로 해당 노드에서 Docker Container 프로세스를 정지

# oc get pod -n [프로젝트명] -o wide

ex) oc get pod -n mojdev -o wide

 

- Terminating NOD에 접속하여 Docker 확인

# docker ps | grep [프로젝트명]

ex) docker ps | grep mojdev

# docker stop [컨테이너ID]

ex) docker stop 28b4d40a6fd8

# docker rm [컨테이너ID] 또는 docker kill [컨테이너ID]

ex) docker rm 28b4d40a6fd8 또는 docker kill 28b4d40a6fd8
```

### 21. POD 내 컨테이너 메모리, CPU 확인
```
# oc get pod -n [프로젝트명] -o wide

# oc rsh [pod명]

# top

# oc adm top node
```

### 22. Openshift Git 형상 주소 변경 시 작업
```
1. 프로젝트 내 배포 설정 정보 확인

# oc get bc -n wizard

 
# oc edit bc -n [프로젝트명] [배포설정명]

ex) oc edit bc -n test01 wizard

source:
    git:
      ref: master
      uri: http://xxx:xxx@xxx.xxx.xxx.xxx:8443/scm/git/openshift_portal
    type: Git
```

### 23. 템플릿 조회
```
# oc get templates -n openshift

# oc get templates -n {프로젝트명}

# oc describe templates {템플릿명} -n {프로젝트명}

```

### 24. 템플릿 어플리케이션 생성
```
# oc new-app -template={템플릿명} -p APPLICATION_NAME={어플리케이션명} -p APPLICATION_DOMAIN={“어플리케이션 도메인명”} -p SOURCE_REPOSITORY_URL={GIT_URL}
```

### 25. 프로젝트 어플리케이션 볼륨 추가
```
# oc set volume dc/{배포설정명} --add --name={PV명} -t pvc --claim-name={PVC명} –m {Mount Point} --overwrite
```

### 26. 프로젝트 어플리케이션 빌드설정 조회
```
# oc get bc -n {프로젝트명} 또는 oc get bc
```

### 27. 프로젝트 어플리케이션 빌드/배포 요청
```
# oc start-build {빌드명} -n {프로젝트명}

: bulid 수행

# oc start-build {빌드명} --follow –n {프로젝트명}

: 빌드 수행 및 로그 확인

# oc start-build {빌드명} --from-build={빌드명-빌드번호} -n {프로젝트명}

: 특정 빌드번호로 빌드수행(oc start-build --from-build=wizard-10 –n mojdev)
```

###  28. 프로젝트 어플리케이션 빌드로그
```
# oc logs -f {빌드POD명} -n {프로젝트명} 또는 oc logs -f {빌드POD명}
```

### 29. 프로젝트 어플리케이션 POD 컨테이너 롤백
```
# oc rollback {배포설정명}

: 마지막으로 성공한 버전으로 rollback 수행

# oc rollback {배포설정명} --to-version= {버전번호} --dry-run

: 실 rollback이 수행되지 않고, 지정된 버전의 rollback의 결과를 사전 확인

# oc rollback {배포설정명} --to-version= {버전번호}

: 지정된 버전으로 rollback 수행

# oc deploy {배포설정명} --enable-triggers -n {프로젝트명}
```


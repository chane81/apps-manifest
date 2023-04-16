# application manifest

- helm + kustomize 조합으로 메니페스트 관리를 합니다.
- main 브랜치만 운영을 합니다.

## 폴더별 설명

- helm

  - frontend 와 backend 리소스 생성에 사용할 공용 custom helm 템플릿이 위치합니다.

- apps

  - frontend 와 backend 의 각 환경별(dev, staging, prod)로 생성해야할 리소스에 대한 메니페스트를 정의합니다.
  - 각 환경별(dev, staging, prod)는 서로 다른 vpc에 있는 eks 에 배포가 됩니다.
  - kustomize 를 통해 관리를 하고, helm 폴더에서 공용템플릿을 사용하여 메니페스트 정의를 합니다.
  - argocd 에서 해당 apps/환경/어플리케이션 폴더를 계속 watch 를 하고 있다가 메니페스트의 변경이 감지되면 배포가 진행됩니다.
  - 각 어플리케이션의 github action 이 수행되면서 kustomization.yaml 의 image.tag 가 변경이되면 배포가 진행됩니다.

- apps 폴더

  - 1 depth 로 각 환경별 폴더가 있으며 하위에 각 어플리케이션별 폴더가 위치합니다.
  - 어플리케이션 폴더 하위에는 예를들어 lake-admin-web 폴더 하위에 있는 original 은 develop 브랜치에서 push가 발생시 해당 폴더의 manifest 를 바라보다 배포를 합니다.
  - unit/~하위(information, report등) 에서 각 하위 폴더는 유닛별로 생성하는 깃헙 브랜치입니다.
  - unit 브랜치는 dev 환경의 서브브랜치이므로 prod 환경에서는 unit 폴더가 존재하지 않습니다.
  - 아래 폴더별 브랜치 매칭입니다.

    - dev

      - lake-admin-web
        - original <--> develop
        - unit/information <--> unit/information
        - unit/settlement <--> unit/settlement
      - lake-api
        - original <--> develop
          ...

    - prod
      - lake-admin-web <--> prod
      - lake-api <--> prod

## kustomize

- github action 에서 kustomize 파일의 image repo 주소와, tag 정보를 update 하여 자동 커밋 & 푸시합니다.
- 아래 yaml 부분에서 newName 은 ecr repo 주소로 바뀌고, newTag 부분은 ecr에 올라간 이미지 태그명으로 바뀝니다.
- change 예시
  - newName: "image-url" => "xxxxxxxxx.dkr.ecr.ap-southeast-1.amazonaws.com/kr-lake-unit-dev"
  - newTag: "image-tag" => "2f4b676_20230421100313"

```yaml
# kustomize overlay
images:
  - name: overlay-image
    # newName: 111113151111.dkr.ecr.ap-southeast-1.amazonaws.com/kr-lake-unit-dev
    # newTag: 2f4b676_20230421100313
    newName: image-url
    newTag: image-tag
```

## Command

- 각 어플리케이션별 kustomize build 및 yaml 확인

```bash
  # application 폴더로 이동
  $ cd apps/dev/lake-admin-web

  ## build 및 템플릿 생성하여 확인
  $ kustomize build --enable-helm . > temp.yaml
  $ kubectl kustomize --enable-helm . > temp.yaml

  $ kubectl apply -f temp.yaml
```

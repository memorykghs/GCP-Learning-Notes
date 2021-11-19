 * Container Registry 和 Artifact Registry 有什麼差別?
   Artifact Registry 比較廣義，除了容器鏡像外，也可以放nodejs module, jar lib等
   在虛擬機的世界，build出來的binary 也是一種Artifact 可以放在Artifact registry
 
   * https://cloud.google.com/blog/products/application-development/understanding-artifact-registry-vs-container-registry

* [IaC](https://www.trendmicro.com/zh_tw/what-is/cloud-security/infrastructure-as-code.html)
* https://github.com/GoogleContainerTools/container-structure-test
* Java Lint
  * Java Linter：VS code插件 https://code.visualstudio.com/docs/java/java-linting
  * 2021的top 10 Java Linter: https://lightrun.com/java/top-10-java-linters/
<br/>

* 檢查工具：https://github.com/GoogleContainerTools/container-structure-test 
* Docker Desktop 替代工具：[Docker-compatible CLI for containerd](https://github.com/containerd/nerdctl)
  * Docker 其實可以通過設定一個 DOCKER_HOST 使用遠端的 docker service
  * 目前無論用 podman or nerdctl 都會有一個問題，許多CI/CD工具可能尚未整合，好的CI/CD可以用bash script的可以自己客製
  * 但是若沒有辦法客制command的，就可以通過設定 DOCKER_HOST 讓這個docker build發生在一個集中的server上
<br/>

* for docker build 的 purpose, 包括 shawn 說的工具大致可以整理如下：
  * Google Friendly
  1. Google Cloud Build
  `$ gcloud builds submit -t gcr.io/my-project/my-image .`

  * 在 k8s 裡 Build 
  1. Tekton
  2. kaniko (Dockerfile)

  * 外部工具
  buildah / podman / nerdctl

  * Language specific
  1. ko (go)
  2. jib (java)
  3. nodejs-container-image-builder (nodejs)

* Proxy 工具
  * [OWASP ZAP zed attack proxy](https://owasp.org/www-project-zap/)

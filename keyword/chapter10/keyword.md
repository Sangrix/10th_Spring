- 클라우드 컴퓨팅이란?
    
    > 클라우드 컴퓨팅이란, 인터넷을 통해 서버, 스토리지, 데이터베이스, 네트워킹, 소프트웨어 등 컴퓨팅 자원을 온디멘드로 주문하여 사용하고 사용한 만큼만 비용을 지불하는 기술
    > 
    
    **[등장 배경]**
    
    - **비즈니스 환경의 급격한 변화**: 새로운 서비스나 스타트업이 HW 장비를 구매하고 데이터 센터를 구축하는데 너무 오랜 시간이 걸리는 것에 비해 시장 변화는 빨라져서 이에 대응하기 위한 즉각적인 인프라 확보가 필요해졌다.
    - **인프라 관리 비용 절감**: 과거에는 트래픽이 몰리는 피크 타임을 기준으로 서버를 구입해야 하였는데, 이로 인해 평상시에는 서버 자원이 유휴 상태로 방치되는 문제가 있었다.
    - **가상화 기술의 발전**: 하나의 물리적 서버를 여러 개의 가상 서버로 쪼개어 효율적으로 사용할 수 있는 가상화 기술이 성숙하면서, 클라우드 서비스의 기술적 기반이 마련되었다.
    
    **[장단점]**
    
    **장점**
    
    - **초기 투자 비용의 절감**: 비싼 HW 자원을 선제적으로 구매할 필요가 없어, 대규모 자본 지출을 운영 비용으로 전환 가능하다.
    - **유연성 및 확장성**: 급격한 트래픽 변동에 유연하게 대처할 수 있어, 서버의 다운 타임을 최소화하고 안정적인 서비스를 제공할 수 있다.
    
    **단점**
    
    - **인터넷 의존성**: 인터넷 연결이 끊기거나 클라우드 제공업체의 네트워킹 장애가 발생하면 서비스 전체가 마비될 수 있다.
    - **클라우드 종속성**: 특정 클라우드사의 전용 서비스나 아키텍처에 깊게 의존할 경우, 다른 클라우드나 자체 서버로의 이전이 까다롭고 비용이 많이 든다.
    
- AWS? GCP?
    
    > 클라우드 컴퓨팅의 양대 산맥으로 대표적인 클라우드 서비스 제공업체
    > 
    
    **[AWS]**
    
    아마존에서 제공하는 클라우드 서비스로 클라우드 시장에서 점유율 1위를 차지하고 있다.
    
    2006년에 서비스를 시작하여 오랜 역사를 지니고 있으며, 다양한 서비스의 종류와 기능을 가진다.
    
    **[장 단점]**
    
    **장점**
    
    - 구글링이나 커뮤니티를 통해 다양한 레퍼런스를 찾기 용이하다.
    - EC2, RDS, ECS 외에도 AI, IoT등 다양한 서비스를 제공한다.
    - 전 세계에 가장 많은 리전을 보유하고 있다.
    
    **단점**
    
    - 기능이 너무 많고 UI가 복잡하여 학습 곡선이 높은 편이다.
    - 세부적인 설정이 복잡해서 예상보다 많은 비용이 청구될 수 있다.
    
    **[GCP]**
    
    구글에서 제공하는 클라우드 서비스로 구글이 쌓은 강력한 인프라 기술을 바탕으로 제공한다.
    
    AWS에 비해 후발 주자이지만, 인공지능, 컨테이너 기술, 오픈소스 친화적인 강점을 가진다.
    
    **[장 단점]**
    
    **장점**
    
    - 대용량 데이터 분석 툴인 BigQuery는 높은 성능을 가지며, 텐서플로우 기반의 AI 인프라가 잘 구축되어 있다.
    - GCP의 쿠버네티스 엔진의 완성도가 높다.
    - 직관적인 UI와 네트워크 속도가 안정적이다.
    
    **단점**
    
    - AWS에 비해 상대적으로 레퍼런스 자료가 부족하다.
    - 대기업 대상의 커스텀 지원이나 기업형 생태계의 다양성은 AWS에 비해 좁은 편이다.
    
- 환경변수 처리 방법과 왜 환경변수로 민감 정보를 가려야 하는가?
    
    > DB의 비밀번호, API 키, JWT secret key 등 외부로 유출되면 안되는 민감 정보를 다루기 위함이다.
    > 
    
    **민감정보를 가려야 하는 이유**
    
    - Git과 GitHub를 통한 소스코드 유출 방지
    - 개발환경과 운영 환경의 분리 - 배포 유연성
    
    로컬 PC에 설치된 테스트용 DB 연결: `localhost:3306`
    
    AWS RDS 등 실제 클라우드 DB 연결: `prod-db.amazonaws.com`
    
    → 환경 변수를 사용하지 않으면 배포 시마다 코드를 수정해야 하거나, 환경별로 코드를 다르게 관리해야 하는데, 코드를 그대로 두고 서버의 설정값만 바꾸어 유연한 대처가 가능하다.
    
    - 권한 분리 및 보안성 강화
    
    **환경변수 처리 방법**
    
    1. 로컬 개발 환경에서 `.env`파일 활용하기
    
    `.env`라는 이름의 파일을 만들고 KEY=VALUE 형태로 민감 정보를 저장한다.
    
    ```markdown
    # .env 파일 예시
    DB_HOST=localhost
    DB_USER=root
    DB_PASSWORD=my_super_secret_password_1234
    JWT_SECRET_KEY=highly_secure_random_string_xyz
    ```
    
    <aside>
    📌
    
    이때, 반드시 .gitignore 파일에 .env 파일이 등록되어야 한다.
    
    </aside>
    
    대신, 팀원들이 어떤 환경변수를 필요로 하는지 확인할 수 있도록 `.env.example`같은 가짜 파일을 만들어 공유하는 것이 관례적이다.
    
    1. 프레임워크에서 환경변수 읽기
    
    `application.yml` 또는 `application.properties`에서 `${...}` 문법을 사용해 시스템 환경변수를 주입받는다.
    
    ```markdown
    # application.yml 예시
    spring:
      datasource:
        url: jdbc:mysql://${DB_HOST}:3306/my_db
        username: ${DB_USER}
        password: ${DB_PASSWORD}
    ```
    
    1. 운영 서버에 환경변수 설정하기
    - **Docker Compose:** `environment` 옵션이나 호스트의 환경변수를 그대로 넘겨준다.
    - **AWS (EC2, ECS 등):** AWS Systems Manager의 **Parameter Store**나 **Secrets Manager** 같은 안전한 보관소를 활용하거나, 컨테이너 설정 내부의 Environment 변수 탭에 직접 주입한다.
    - **CI/CD (GitHub Actions):** 레포지토리 설정의 `Settings -> Secrets and variables -> Actions`에 값을 저장해 두고, 빌드/배포 스크립트(`workflow.yml`)에서 `${{ secrets.DB_PASSWORD }}` 형태로 꺼내 씁니다.
    
- yml 환경 분리 방법
    
    
    **파일 분리 방식**
    하나의 큰 파일 대신, 환경별로 별도의 `.yml` 파일을 생성하는 방식입니다. 파일명이 `application-{profile}.yml` 형식을 따르면 Spring Boot가 자동으로 인식한다.
    
    **파일 구조**
    
    프로젝트의 `src/main/resources` 디렉토리 아래에 이러한 이름의 파일을 만든다.
    
    - `application.yml` : 공통 설정 및 기본 활성화할 프로필 지정
    
    ```markdown
    spring:
      profiles:
        active: local # 아무런 설정을 주지 않으면 기본적으로 local 프로필을 활성화함
    
      # 모든 환경에서 공통으로 사용하는 설정 (예: JPA 설정 등)
      jpa:
        hibernate:
          ddl-auto: none
    ```
    
    - `application-local.yml` : 로컬 개발용 설정 (내 PC의 DB 등)
    
    ```markdown
    spring:
      datasource:
        url: jdbc:mysql://localhost:3306/local_db
        username: root
        password: local_password
    
    # 로컬에서는 SQL 로그를 보기 위해 ddl-auto를 update로 설정 가능
      jpa:
        hibernate:
          ddl-auto: update
        show-sql: true
    ```
    
    - `application-dev.yml` : 개발 서버(테스트 서버)용 설정
    - `application-prod.yml` : 실제 운영 서버(AWS RDS 등)용 설정
    
    ```markdown
    spring:
      datasource:
        # 민감 정보는 앞서 배운 환경변수(${...})를 사용해 외부에서 주입받음
        url: jdbc:mysql://${PROD_DB_HOST}:3306/prod_db
        username: ${PROD_DB_USER}
        password: ${PROD_DB_PASSWORD}
    
      jpa:
        hibernate:
          ddl-auto: validate
        show-sql: false
    ```
    
    **하나의 파일 내에서 구분하는 방법**
    Spring Boot 2.4 버전 이후부터는 `---` (점선 3개) 구문과 `spring.config.activate.on-profile`을 사용하여 **하나의 `application.yml` 파일 안에서** 환경을 나눌 수 있다.
    
    ```markdown
    # 1. 공통 설정 영역
    spring:
      profiles:
        active: local # 기본값
      jpa:
        open-in-view: false
    
    ---
    # 2. 로컬 환경 영역
    spring:
      config:
        activate:
          on-profile: local
      datasource:
        url: jdbc:mysql://localhost:3306/local_db
        username: root
        password: local_password
    
    ---
    # 3. 운영 환경 영역
    spring:
      config:
        activate:
          on-profile: prod
      datasource:
        url: jdbc:mysql://${PROD_DB_HOST}:3306/prod_db
        username: ${PROD_DB_USER}
        password: ${PROD_DB_PASSWORD}
    ```
    
    **서버를 실행할 때 특정 환경 적용하는 방법**
    
    **Jar 파일 직접 실행시**
    터미널에서 애플리케이션을 구동할 때 `-Dspring.profiles.active` 옵션을 인자로 넘겨준다.
    
    ```markdown
    # 운영 서버에서 실행할 때
    java -jar -Dspring.profiles.active=prod my-project.jar
    
    # 개발(테스트) 서버에서 실행할 때
    java -jar -Dspring.profiles.active=dev my-project.jar
    ```
    
    **IntelliJ IDEA (로컬 개발 환경)**
    
    - 우측 상단의 실행 구성(Run Configuration) 드롭다운을 클릭하고 Edit Configurations...로 들어갑니다.
    - **Active profiles** 항목에 `local` 혹은 `dev`를 입력하고 저장합니다.
    - 실행(Run)하면 해당 프로필의 yml 설정이 적용됩니다.
    
    **Docker Compose 활용시**
    Docker 컨테이너로 띄울 때는 `environment` 옵션에 프로필 키를 환경변수로 넣어주면 됩니다.
    
    ```markdown
    version: '3.8'
    services:
      backend-service:
        image: my-backend-image:latest
        ports:
          - "8080:8080"
        environment:
          - SPRING_PROFILES_ACTIVE=prod
          - PROD_DB_HOST=aws-rds-endpoint...
          - PROD_DB_USER=admin
          - PROD_DB_PASSWORD=secret_password
    ```
    
- Docker와 .jar vs Docker 이미지
    
    > 백엔드 애플리케이션을 배포할 때 `Docker 컨테이너 안에서 .jar를 실행하는 방식`과 `아예 애플리케이션을 포함한 Docker 이미지 자체를 빌드하여 관리하는 방식`의 차이점
    > 
    
    현대적인 DevOps 및 클라우드 배포 환경에서는 “애플리케이션을 포함한 완전한 Docker 이미지”를 만들어서 배포하는 것이 표준이다.
    
    **방식 A: 가벼운 Docker 컨테이너 + 외부 `.jar` 주입**
    
    이 방식은 Java(JDK) 환경만 셋팅된 공통 Docker 이미지를 띄워두고, 배포할 때마다 빌드된 `.jar` 파일을 컨테이너 내부로 링크(마운트)하거나 복사해서 실행하는 방식이다.
    
    - **특징:** Docker 컨테이너는 '자바 실행기' 역할만 하고, 소스코드가 컴파일된 `.jar` 파일은 컨테이너 외부(호스트 OS)에서 관리한다.
    - **장점:** `.jar` 파일만 바꾸면 되므로 이미지 빌드 시간이 따로 걸리지 않아 배포 속도가 미세하게 빠를 수 있다.
    - **단점:** 컨테이너와 `.jar` 파일이 분리되어 있어, 서버 환경의 독립성과 이식성이라는 Docker 본연의 장점이 퇴색됩니다. 여러 대의 서버에 배포할 때 `.jar` 파일을 각각 전달해야 하는 번거로움이 있다.
    
    **방식 B: 애플리케이션을 포함한 Docker 이미지 자체를 빌드**
    
    Spring Boot 코드가 수정되면 가상 머신 스냅샷을 찍듯 **`JDK 환경 + .jar 파일`을 통째로 캡슐화하여 하나의 새로운 Docker 이미지**를 만드는 방식이다.
    
    - **특징:** 이미지 자체에 `app.jar`가 내장되어 있습니다. 이 이미지를 Docker Hub나 AWS ECR 같은 레지스트리에 올려두면, 어떤 서버에서든 `docker pull` 한 줄로 완벽히 동일한 환경을 구동할 수 있다.
    - **장점:** 배포 단위가 '이미지' 하나로 단일화되므로 관리가 매우 깔끔하고, 버전 관리(롤백)가 용이하다.
    - **단점:** 코드가 바뀔 때마다 Docker 이미지를 다시 빌드하고 푸시해야 하므로 빌드 시간이 조금 더 소요된다.
    
    **Dockerfile 작성법**
    
    ```markdown
    # 1. Base 이미지 지정 (JDK 환경)
    FROM openjdk:17-jdk-slim
    
    # 2. 작업 디렉토리 설정
    WORKDIR /app
    
    # 3. 빌드된 jar 파일을 컨테이너 내부로 복사 (내장화)
    COPY build/libs/my-app-0.0.1-SNAPSHOT.jar app.jar
    
    # 4. 애플리케이션 실행 명령어
    ENTRYPOINT ["java", "-jar", "app.jar"]
    ```
    
    이렇게 만든 Dockerfile을 기반으로 `docker build -t 내이미지이름:버전 .` 명령을 실행하여 **하나의 완성된 독립된 이미지**를 만든다.
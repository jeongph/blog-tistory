![https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b03a8beb-9081-4237-a2e0-18b3050f3f68/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230126%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230126T151828Z&X-Amz-Expires=86400&X-Amz-Signature=a8436dcfbb39c52faf1729a04f0fdc1b310e530f57f93bb5dc3b64bea7f13be7&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b03a8beb-9081-4237-a2e0-18b3050f3f68/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230126%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230126T151828Z&X-Amz-Expires=86400&X-Amz-Signature=a8436dcfbb39c52faf1729a04f0fdc1b310e530f57f93bb5dc3b64bea7f13be7&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

> 코로나라는 핑계로 잃어버린 시간들.. 2023은 “Just Fxxking Do It”
> 

# 에세이

- [경지에 오르는 세 번째 관문](https://brunch.co.kr/@osyvv/105) (2020/02/18)
    - 어떤 경지에 오르기까지 시작, 지속, 재미(칭찬) 을 거치며, 실력을 키우고 지속하다보면 경지에 오를 수 있다는 짧은 글
    - 예전에 저장해놓고 단순히 “일리가 있네” 라고 생각했던 글인데, 최근에 비교적 “시작의 힘”을 더 뼈저리게 느끼고있고 일단 시도하는것과 싫어도 지속하는것, 작은 스스로의 칭찬과 보상들이 계속 모여서 하나의 성공을 이룬다는것을 책으로 많이 배웠고, 약 2년간의 운동으로 실제로 느끼고있다

# 개발 관련

- [개발자라면 마주치는 기술 부채, 꼭 다 갚아야 하나요?](https://yozm.wishket.com/magazine/detail/1331/) (2022/02/17)
    - 개발을 하면서 불가피하게 발생하는 개술부채에 대하여 고민하고, 어떻게하면 최소화할 수 있을지 공유한 글
    - 비교적 예전엔 나도 “어떻게하면 최신의 기술을 사용하고 적용할 수 있을까” 에 대한 고민을 강박적으로 하고 뒤쳐지면 안될것같은 느낌을 많이 받았는데, 함께 일하는 사람들이 늘어나며 러닝커브를 함께 감당해야하고, “어떤게 회사에 더 도움되는 개발일까” 에 대한 고민을 하고 “그때는 그게 맞았다” 라는 생각을.. 뭐 여러 글이나 책을 통해 간접경험하고 고민하게 도와준 글 중 하나고, 사실 내 생각엔 소통도 중요하고 문서화나 정보공유도 중요한 것 같다
- [BDD? SDD? 팀 프로젝트 다 같이 개발∙설계하는 방법](https://yozm.wishket.com/magazine/detail/1565/) (2022/07/06)
    - BDD 와 SDD 방법론을 활용하여 단순히 개발직군만이 아닌, 여럿이서 함께 프로젝트를 개발할 때 생산성을 높일 방법을 가이드하는 글
    - [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) 는 [TDD](https://ko.wikipedia.org/wiki/%ED%85%8C%EC%8A%A4%ED%8A%B8_%EC%A3%BC%EB%8F%84_%EA%B0%9C%EB%B0%9C) 때문에 테스트를 작성하면서 들어봤지만, [SDD](https://medium.com/@hintology/sdd-schema-driven-development-f1d232d73ea6) 는 조금 생소한 방법론이었는데, 회사라는 공동체에서 불가피하게 하나의 프로젝트를 여러사람, 여러 직군이 참여하여 함께 개발하다보니 “어떻게하면 다른 직군과 협업하면서 생산성을 높일 수 있을까?” 에 대한 고민을 다들 하고있고, 이런 고민이 있다면 읽어볼만한 글
- [[Tecoble] DTO의 사용 범위에 대하여](https://xlffm3.github.io/spring%20&%20spring%20boot/DTOLayer/) (2021/05/22)
    - MVC 패턴을 사용하는 프레임워크에서 데이터를 전달할때 많이 사용하는 DTO(data transfer object) 를 어디까지 사용해야할지 고민하는 글
    - Entity 자체를 End-point 바깥까지 공개해버리면 내부 도메인의 형태가 공개되어버리고, 간단한 변경사항이 생겨도 Entity 코드를 수정해야하는 아이러니한(?) 상황이 발생해서, 습관적으로 DTO 를 사용하라고 알려주지만, 반대로 DTO 의 무분별한 사용이 비즈니스 로직이나 Persistence Layer 까지 넘어와서 구조 자체를 무너뜨리는 경우를 보곤 종종 안타까웠는데, 이런 고민을 했다면 읽어볼만한 글
- [단 하나의 API 사이트를 위한 여정 - Part 1](https://blog.payhere.in/tech-220520/) (2022/05/20)
    - 페이히어 CTO 안성현 님의 [MSA](https://ko.wikipedia.org/wiki/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4) 구조의 여러개의 서비스들의 API 를 어떻게 문서화하면 좋을지 고민하고 해결을 시도해보는 글
    - 요새는 조금 회의적이지만, 이미 많은 서비스들이 모놀리틱 구조에서 마이크로서비스 구조로 변경되었고, 그 과정에서 자연스럽게 분할되는 API 문서들을 (e.g. Swagger) 어떻게 관리하면 좋을지 고민하곤 했는데, Swagger 특성상 여러개의 Docket 을 사용하여 (원랜 버전관리 하라고 만든거지만..) 각각의 서비스를 하나의 Swagger 로 보여주고 사용했지만 여전히 “보안” 이라는 측면에서 “오픈소스도 아닌 우리 회사 API를 어떻게 감추면 좋을까” 고민했었기 때문에 재밌었던 글
- [일 잘 하는 개발자는 왜 비즈니스까지 신경쓸까?](https://yozm.wishket.com/magazine/detail/1189/) (2021/11/29)
    - 개발자가 단순히 “개발을 잘하는” 범위를 넘어서, 비즈니스나 기술의 니즈에 대해 파악하고 설계에 대해 고민하고, 중요한것이 무엇인지 아는것에 대해 알려주는 글
    - 이제 단순히 “개발만 잘하는 것” 에 대한 메리트는 거의 없어졌다고 생각하고, 코더가 아닌 개발자, 혹은 아키텍터나 프로젝트 매니징의 관점에서 프로젝트를 대해야 더 의미있는 역할이 된다고 생각하고, 단순히 [copilot](https://github.com/features/copilot) 같은 것만 봐도 결국 “똑같은 일을 하는 사람”은 기계로 치환될 수 있고, “고민하는 사람”은 기계를 설계한다고 생각한다

# 기타

- [기술 블로그(Engineering Blog)란?](https://www.44bits.io/ko/keyword/engineering-blog) (2021/02/14)
    - 기술블로그란 무엇이고, 기술블로그를 위한 어떤 서비스들이 존재하는지 간략하게 설명해주는 글이고 아래 예시로 한국 IT 기업의 기술블로그들과 [https://techblogposts.com/](https://techblogposts.com/) 사이트를 소개하고있는데, 나 또한 이 tech-blog-posts 를 잘 사용하고 있기에 도움이된 글 (블로그 서비스를 뭘 이용할까 고민을 엄청했지만 결국 “일단 시작하기” 와 “그래도 코드랑은 나누고싶은데” 를 빠르게 실행하기위해 티스토리를 사용중이라)

# [스프링 프레임워크 핵심기술] - @Component와 컴포넌트 스캔

---

> 백기선님의 **스프링 프레임워크 핵심 기술**이라는 강좌를 들으며 공부한 내용을 정리한 글입니다.
>

이전에 [ApplicationContext와 다양한 빈 설정 방법](https://www.notion.so/ApplicationContext-458145cdcb9d4646b3b743de5b21697b) 글에서 잠깐 다루었던 내용 중

@ComponentScan과 @Component의 기능과 동작 원리를 조금 더 자세히 살펴보자.

# 1. @ComponentScan

@ComponentScan은 스프링 3.1부터 도입된 Annotation이며 **스캔 위치를 설정**하고,

어떤 Annotation을 스캔할지 또는 하지 않을지 결정하는 **Filter 기능**을 가지고있다.

중요한 설정으로는 **BasePackages**와 **BasePackageClasses**가 있다. 이는 스캔 위치를 설정하는

방법들인데, BasePackages는 String으로 입력된 패키지의 경로를 스캔하는 ****방법이고

BasePackageClasses는 전달된 **클래스 기준**으로 스캔을 시작하는 방법이다.

String을 입력받는 BasePackages는 TypeSafe하지 않기 때문에 **BasePackageClasses** 사용을 조금 더

추천한다.

## 1. BasePackageClasses

@ComponentScan이 붙어있는 Configuration부터 스캔을 시작한다.

Spring boot 프로젝트에서는 @SpringBootApplication에 이미 @Configuration과 @ComponentScan

이 붙어있다. 따라서 @SpringBootApplication 기준으로 스캐닝이 시작된다.

해당 Application이 위치한 패키지 이하의 모든 클래스에 컴포넌트 스캔이 진행된다.

**하지만 이 패키지 밖에있는 클래스들은 스캔이 안된다.**

예를들면,

![](https://blog.kakaocdn.net/dn/bk517m/btqEbuQ5gTp/siwfXiHsfUd5Mt9d9nzNG1/img.png)

이와 같은 패키지 구조일 때 DemoApplication이 속해있는 패키지는 컴포넌트 스캔이 적용되지만

이 패키지 밖에있는 out의 MyTestService는 스캔이 되질 않는다.

![](https://blog.kakaocdn.net/dn/dEMpm0/btqEcUVzZHU/YkQea17NT00U1Dqyhiug01/img.png)

![](https://blog.kakaocdn.net/dn/b75XJ2/btqEbQM8jd2/YRiLvDkhPTOtu7HldmFicK/img.png)

MyTestService에 @Service가 붙어있음에도 DemoApplication에서 MyTestService의 의존성 주입을 시도하면 bean을 찾을 수 없다는 에러가 발생한다.

그 이유는 MyTestService는 컴포넌트 스캔의 검색 범위에 해당되지 않았고, 그 결과 Bean으로 등록이 되지 않았기 때문이다.

---

# 2. @Component

컴포넌트 스캔이 스캐닝하는 기준이 바로 @Component이다.

@Component를 들고있는 클래스들이 스캐닝되고, Bean으로 등록된다.

대표적인 컴포넌트

- @Controller
- @Repository
- @Service
- @Configuration

---

# 3. 컴포넌트 스캔의 동작 원리

@ComponentScan은 BeanFactoryPostProcessor를 구현한 **ConfigurationClassPostProcessor**에 의해 동작한다.

BeanFactoryPostProcessor는 다른 모든 Bean들을 만들기 이전에 BeanFactoryPostProcessor의

구현체들을 모두 적용한다.

즉, 다른 Bean들을 등록하기 전에 **컴포넌트 스캔**을해서 Bean으로 등록해준다.

이전 글인 [@Autowired](https://www.notion.so/Autowired-27fa94e033c74858afc66930d1645a7b)에서 설명했던 BeanPostProcessor과는 비슷하지만, 실행 시점이 다르다.

쉽게 설명하면,

@ComponentScan은 다른 Bean들을 찾아 BeanFactoryPostProcessor의 구현체를 적용하여 Bean으로 등록해주는 과정이고

@Autowired는 등록된 다른 Bean을 찾아 BeanPostProcessor의 구현체를 적용하여 의존성 주입을

적용하는 역할을 하는 것이다.

여기서 말하는 **다른 Bean**이란 우리가 직접 등록하는 Bean들을 가르킨다.

- @Controller, @Service, @Bean, @Respository 등

---

강의를 듣고 다시 한번 글로 정리해보는게 확실히 많은 도움이 되고 있고 꽤 괜찮은 학습 방법같다.

강의를 들을때는 이해한 것 같아도, 제대로 이해하지 못하거나 애매하게 이해한 부분은 확실히

글로 옮기기 어려웠다. 글을 쓰기위해 강의를 몇 번씩 돌려보다 보면 점점 이해가 되고 강의만으로

이해가 어려운 부분은 따로 찾아보기도 하는 과정을 거치면서 공부한 내용이 조금 더 머릿속에

잘 남는 것 같다.
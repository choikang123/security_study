### 인증 서버로 부터 정보를 받아 회원정보를 저장할때 고유 socialId가 있는데, email로 확인해서 저장하는지 궁금함이 생겨 찾아보게 되었다.
### orElseGet의 사용 방법을 사용하는 이유가 궁금하여 추가로 찾아보게 됨.

```java
// 회원 정보 조회 또는 저장 메서드
private Member getOrSave(OAuthAttributes oauthAttributes) {
    return memberRepository.findByEmail(oauthAttributes.getEmail())
            .orElseGet(() -> memberRepository.save(oauthAttributes.toEntity()));
}
```

## 회원 조회 방식과 orElseGet() 설명

### 이메일 vs 소셜 ID로 사용자 찾기

해당 코드에서는 이메일로 회원을 찾고 있는데, 이렇게 하는 이유는:

- 계정 통합을 위해서: 다양한 소셜 로그인(카카오, 구글 등)을 사용하더라도 같은 이메일이라면 하나의 계정으로 통합할 수 있다는 장점을 가진다.
- 즉 사용자가 카카오로 가입했다가 다음에 구글로 로그인해도, 이메일이 같다면 동일 계정으로 인식
- 일반 로그인과 통합: 이메일/비밀번호 로그인과 소셜 로그인을 모두 지원하는 경우 계정 연동이 용이

### 소셜 ID로 찾아도 될까요?

```java
private Member getOrSave(OAuthAttributes oauthAttributes) {
    return memberRepository.findBySocialId(oauthAttributes.getSocialId())
            .orElseGet(() -> memberRepository.save(oauthAttributes.toEntity()));
}
```

소셜 ID로 찾는 것도 가능하지만 더 정확한 사용자 식별이 가능하지만 다른 소셜 서비스로 로그인하면 별도 계정이 생성되기 때문에 이메일로 중복 가입을 막는게 중요하다면
이메일로 찾는것이 좋지만 상황에 따라 달라질 거 같다

## orElseGet() 메소드 설명

orElseGet()은 Optional 클래스의 메소드로:

**기능**: 값이 존재하면 그 값을 반환하고, 값이 없으면(null) 매개변수로 전달된 함수를 실행

**구문 분석**:

```java
.orElseGet(() -> memberRepository.save(oauthAttributes.toEntity()))
```

- 회원이 존재하면: 찾은 회원 반환
- 회원이 없으면: () 안의 코드 실행 (새 회원 생성 후 저장)

즉 orElseGet()은 필요할 때만 안의 코드가 실행(회원이 없을 때만) 그렇다면 orElse()와의 차이점은 무엇인가?

orElse()는 값의 존재 여부와 상관없이 항상 매개변수를 평가

```java
// 항상 newMember()가 실행됨 (불필요한 객체 생성 가능)
member.orElse(newMember())

// member가 없을 때만 newMember()가 실행됨
member.orElseGet(() -> newMember())
```

이런 차이로 인해 비용이 큰 연산이나 DB 저장 작업에는 orElseGet()이 권장됨


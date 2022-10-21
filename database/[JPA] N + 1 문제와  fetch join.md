## Fetch join이란

fetch join이란 JPQL에서 제공하는 성능 최적화 기능으로 SQL 
조인의 종류가 아니기 때문에 JPA를 처음 접하시는 분들이 많이 어려워 
하시는 개념입니다.

fetch join은 다음과 같은 문법으로 사용되며 연관된 엔티티 `(xxxToOne)`, 컬렉션 `(xxxToMany)` 을 SQL 한 번에 함께 조회
(한방 쿼리) 하는 기능입니다.

- 페치 조인 :: = [LEFT[OUTER] | INNER] JOIN FETCH 조인 
경로
    
페치 조인을 사용하여 회원을 조회하면서 연관된 팀도 함께 한방 쿼리로 
조회하는 예를 살펴보겠습니다.

```java
select m from Member m join fetch m.team
```

![Untitled](https://i.imgur.com/KarygTF.png)

위의 jpql은 다음과 같은 SQL문으로 번역되어 나가게 됩니다.

```java
SELECT M.*, T.* FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```

![Untitled](https://i.imgur.com/5wIqGxs.png)

jpql과 SQL문의 차이점을 보면 jpql의 경우 `m` 만 프로젝션 했지만 
회원과 팀 정보 모두 조회합니다.

영속성 컨텍스트(1차 캐시)에는 총 5개의 엔티티가 보관됩니다.

![Untitled](https://i.imgur.com/f0jJ6za.png)

## N + 1 문제 - ManyToOne

JPA를 사용하면 반드시 한번쯤은 만나는 N + 1 문제는 페치 조인으로 
해결 가능한 대표적인 문제입니다.

`다대일` 관계인 TeamMember 엔티티와 Team 엔티티가 존재합니다.

다음과 같이 TeamMember 엔티티에서 Team 엔티티를 지연로딩으로 
설정한 경우 발생하는 N + 1 문제의 예를 들어보도록 하겠습니다.

`TeamMember.java`

![스크린샷 2022-05-12 오후 3.22.45.png](https://i.imgur.com/lz6Sw9p.png)

`Team.java`

![스크린샷 2022-05-12 오후 3.24.09.png](https://i.imgur.com/8ZmDJ9D.png)

각 엔티티 테이블에 저장된 정보는 다음과 같습니다.

- `회원` : 회원1, 회원2, 회원3 ...
- `팀` : 팀A, 팀B, 팀3 ...
- `회원 - 팀` : $($회원1 - 팀A), $($회원2 - 팀B), $($회원3 - 팀C) ...

회원 엔티티를 조회한 다음, 회원의 이름과 회원이 속한 팀의 이름을 
조회하는 로직을 아래와 같이 구현해 보았습니다.

```java
String query = "select tm from TeamMember tm";

List<TeamMember> members = em.createQuery(query, TeamMember.class)
					.getResultList();

for (TeamMember teamMember : teamMembers) {
		System.out.println("teamMember = " + teamMember.getUsername() + ", " + "teamName = " + teamMember.getTeam().name());
}
```

지연로딩으로 설정해 주었기 때문에 teamMember 엔티티를 조회하면서 
team 엔티티를 프록시 객체로 생성하여 가져옵니다.

```java
String query = "select tm from TeamMember tm";
```

> **현재 총 쿼리 수**
> TeamMemer 조회 : 1
> Team 조회 : 0
> 총 쿼리 : 1 + 0 = 1
    

그 다음 반복문을 돌면서 teamMembers에서 회원 개개인의 정보를 
조회해 옵니다.

```java
for (TeamMember teamMember : teamMembers) {
		System.out.println("teamMember = " + teamMember.getUsername() + ", " + 
		"teamName = " + teamMember.getTeam().name());
```

이때 `teamMember.getTeam().name();` 에 의해서 해당 
teamMember가 속해있는 team 프록시 객체는 name 메서드를 
호출하여 실제 Team 엔티티에 접근하게 됩니다. 

teamMember가 `회원1`인 경우 `(회원1 - 팀A)`

- `팀A` 가 영속성 컨텍스트에 있는지 확인합니다.
- 없으므로 조회 쿼리를 DB에 날려서 `팀A` 를 영속성 컨텍스트에 
올립니다.
> **현재 총 쿼리 수**
> TeamMember 조회 : 1
> Team 조회 : 1
> 총 쿼리 : 1 + 1 = 2
    

teamMember가 `회원2`인 경우 `(회원2 - 팀B)`

- `팀B` 가 영속성 컨텍스트에 있는지 확인합니다.
- 없으므로 조회 쿼리를 DB에 날려서 `팀B` 를 영속성 컨텍스트에 
올립니다.
> **현재 총 쿼리 수**
> TeamMember 조회 : 1
> Team 조회 : 2
> 총 쿼리 : 1 + 2 = 3
    

teamMember가 `회원3`인 경우 `(회원3 - 팀C)`

- `팀C` 가 영속성 컨텍스트에 있는지 확인합니다.
- 없으므로 조회 쿼리를 DB에 날려서 `팀C` 를 영속성 컨텍스트에 
올립니다.
> **현재 총 쿼리 수**
> TeamMember 조회 : 1
> Team 조회 : 3
> 총 쿼리 : 1 + 3 = 4
    

... 

만약 회원 데이터가 N개 존재하고 N명의 회원이 속한 팀이 전부 
다르다면 총 쿼리수는 다음과 같습니다.

**회원수가 N명인 경우 총 쿼리 : 1 + N**

여기서 1은 N개의 회원 데이터를 조회하기 위한 쿼리입니다.

N은 각 회원이 속한 팀 정보를 조회하기 위한 쿼리입니다.

## Fetch join을 통한 N + 1 해결

```java
String query = "select tm from TeamMember tm join fetch tm.team";

List<TeamMember> members = em.createQuery(query, TeamMember.class)
					.getResultList();

for (TeamMember teamMember : teamMembers) {
		System.out.println("teamMember = " + teamMember.getUsername() + ", " + 
		"teamName = " + teamMember.getTeam().name());
}
```

위와 같이 페치 조인을 작성하면 DB로 부터 회원 정보를 조회해오면서 
팀 정보도 함께 조회해 오는 한방쿼리를 작성하여 보내게됩니다.

결과적으로 한번의 쿼리만으로 프록시 객체가 아닌 실제 팀 엔티티가 
영속성 컨텍스트에 전부 올라가있게 됩니다.

이후에 회원이 속한 팀 정보를 조회하더라도 영속성 컨텍스트에서 조회가 
가능하기 때문에 추가 쿼리가 발생하지 않게 됩니다.

## 컬렉션 패치 조인 - OneToMany

이번에는 일대다 관계, 컬렉션 페치 조인에 대해서 알아보도록 
하겠습니다.

```java
String query = "select t from Team t join fetch t.teamMembers";

List<Team> teams = em.createQuery(query, Team.class)
					.getResultList();

for (Team team : teams) {
		System.out.println("team = " + team.getName() + "| teamMembers = " + 
		team.getTeamMembers().size());
}
```

**출력 결과**

```bash
team = 팀A| teamMembers = 2
team = 팀A| teamMembers = 2 // 팀A 데이터가 중복!!
team = 팀B| teamMembers = 1
```

일대다 관계의 테이블을 페치 조인으로 조회하면 데이터 중복이 
일어납니다.  

각각의 테이블을 확인해보면 TeamMember에는 `팀A` 에 속한 팀원이 
두명 존재합니다.

![Untitled](https://i.imgur.com/oVDCfGY.png)

이때 TeamMember 테이블과 Team 테이블을 조인하게 되면 다음과 같이 중복된 Team 정보가 조회됩니다.

![Untitled](https://i.imgur.com/hbRpfub.png)

 

이때 `distinct` 를 사용하여 중복을 제거할 수 있습니다.

SQL의 distinct는 중복 컬럼의 모든 속성값이 같아야만 중복이 
제거되지만 jpql에서의 distinct는 2가지 기능을 제공합니다.

1. SQL에 DISTINCT 추가
2. 애플리케이션에서 엔티티 중복 제거 (엔티티의 주소값이 같으면 
속성값이 달라도 중복 제거)
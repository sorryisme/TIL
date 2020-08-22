## Pageable

참고 블로그 : https://ivvve.github.io/2019/01/13/java/Spring/pagination_3/



> @Controller

```java
 public String movePageNoticeList(@PageableDefault Pageable pageable, Model model){
 		Page<Notice> noticeList = noticeService.getNoticeListAll(pageable);
		model.addAttribute("noticeList", noticeList);
   	return "/notice/noticeList";
 }
```

- @pageDefault 애노테이션은 기본 값으로 size 10 , page 0으로 지정되어 있다.



> Service

```java
@Transactional(readOnly = true)
public Page<Notice> getNoticeListAll(Pageable pageable) {

  int page = (pageable.getPageNumber() == 0) ? 0 : (pageable.getPageNumber() - 1);
  pageable = PageRequest.of(page, 10);

  return noticeRepository.findAll(pageable);
}
```

- 사용자가 보려는 페이지는 1부터 시작하기 때문에 ( -1 )해서 page 값을 맞춰준다
- pageRequest로 Pageable 객체 생성



> Thymeleaf

```html
<nav style="text-align: center;">
    <ul class="pagination"
        th:with="start=${T(Math).floor(boardList.number/10)*10 + 1},
                    last=(${start + 9 < boardList.totalPages ? start + 9 : boardList.totalPages})">
        <li>
            <a th:href="@{/boards(page=1)}" aria-label="First">
                <span aria-hidden="true">First</span>
            </a>
        </li>

        <li th:class="${boardList.first} ? 'disabled'">
            <a th:href="${boardList.first} ? '#' :@{/boards(page=${boardList.number})}" aria-label="Previous">
                <span aria-hidden="true">&lt;</span>
            </a>
        </li>

        <li th:each="page: ${#numbers.sequence(start, last)}" th:class="${page == boardList.number + 1} ? 'active'">
            <a th:text="${page}" th:href="@{/boards(page=${page})}"></a>
        </li>

        <li th:class="${boardList.last} ? 'disabled'">
            <a th:href="${boardList.last} ? '#' : @{/boards(page=${boardList.number + 2})}" aria-label="Next">
                <span aria-hidden="true">&gt;</span>
            </a>
        </li>

        <li>
            <a th:href="@{/boards(page=${boardList.totalPages})}" aria-label="Last">
                <span aria-hidden="true">Last</span>
            </a>
        </li>
    </ul>
</nav>
```

- start=${T(Math).floor(boardList.number/10)*10 + 1}  : 현재페이지(number)로 시작페이지 구하는 로직
- last=(${start + 9 < boardList.totalPages ? start + 9 : boardList.totalPages}) : 시작페이지를 통해서 마지막페이지 구하는 로직
- 시작과 끝의 sequence로 page를 출력한다


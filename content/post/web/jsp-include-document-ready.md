---
title: "JSP Include와 document.ready 완전 정리"
date: 2026-01-19
categories:
- web
tags:
- jsp
- jquery
- document-ready
- tiles
- javascript
keywords:
- JSP Include
- document.ready
- Apache Tiles
- jsp:include
author: "hyunwoo"
draft: false
---

JSP의 다양한 Include 방식(`<%@ include %>`, `<jsp:include>`, `<c:import>`, Apache Tiles)에서 jQuery `document.ready`가 어떤 순서로 실행되는지 정리합니다. 실무에서 자주 쓰는 네임스페이스 패턴과 비동기 의존성 처리 패턴도 함께 다룹니다.

<!--more-->

---

## 1. Include 방식별 비교

### 1-1. 지시자 Include (`<%@ include %>`)

**컴파일 타임에 소스 코드를 물리적으로 합침**

```javascript
<%-- ========== parent.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<!DOCTYPE html>
<html>
<head>
    <script src="/js/jquery-3.6.0.min.js"></script>
</head>
<body>

<%@ include file="header.jsp" %>

<main>
    <h1>본문 영역</h1>
</main>

<script>
$(document).ready(function() {
    console.log('parent ready');
});
</script>

</body>
</html>
```

```javascript
<%-- ========== header.jsp ========== --%>
<header>
    <h2>헤더 영역</h2>
</header>

<script>
$(document).ready(function() {
    console.log('header ready');
});
</script>
```

**컴파일 결과 (단일 Servlet으로 합쳐짐)**

```java
public class parent_jsp extends HttpServlet {
    public void _jspService(HttpServletRequest request, HttpServletResponse response) {
        out.write("<!DOCTYPE html>\n");
        out.write("<html>\n<head>\n");
        out.write("<script src=\"/js/jquery-3.6.0.min.js\"></script>\n");
        out.write("</head>\n<body>\n");

        // <%@ include %> - header.jsp 코드가 직접 삽입됨
        out.write("<header>\n<h2>헤더 영역</h2>\n</header>\n");
        out.write("<script>\n$(document).ready(function() {\n");
        out.write("    console.log('header ready');\n});\n</script>\n");

        out.write("<main>\n<h1>본문 영역</h1>\n</main>\n");
        out.write("<script>\n$(document).ready(function() {\n");
        out.write("    console.log('parent ready');\n});\n</script>\n");
        out.write("</body>\n</html>");
    }
}
```

**실행 순서**

```javascript
header ready
parent ready
```

---

### 1-2. 액션 태그 Include (`<jsp:include>`)

**런타임에 출력 결과를 삽입**

```javascript
<%-- ========== parent.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<!DOCTYPE html>
<html>
<head>
    <script src="/js/jquery-3.6.0.min.js"></script>
</head>
<body>

<script>
$(document).ready(function() {
    console.log('1. parent ready');
});
</script>

<jsp:include page="child1.jsp">
    <jsp:param name="title" value="자식1" />
</jsp:include>

<jsp:include page="child2.jsp">
    <jsp:param name="title" value="자식2" />
</jsp:include>

<jsp:include page="child3.jsp">
    <jsp:param name="title" value="자식3" />
</jsp:include>

</body>
</html>
```

```javascript
<%-- ========== child1.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<section>
    <h2>${param.title}</h2>
    <p>자식1 컨텐츠</p>
</section>

<script>
$(document).ready(function() {
    console.log('2. child1 ready');
});
</script>
```

```javascript
<%-- ========== child2.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<section>
    <h2>${param.title}</h2>
    <p>자식2 컨텐츠</p>
</section>

<script>
$(document).ready(function() {
    console.log('3. child2 ready');
});
</script>
```

```javascript
<%-- ========== child3.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<section>
    <h2>${param.title}</h2>
    <p>자식3 컨텐츠</p>
</section>

<script>
$(document).ready(function() {
    console.log('4. child3 ready');
});
</script>
```

**브라우저가 받는 최종 HTML**

```html
<!DOCTYPE html>
<html>
<head>
    <script src="/js/jquery-3.6.0.min.js"></script>
</head>
<body>

<script>
$(document).ready(function() {
    console.log('1. parent ready');
});
</script>

<section>
    <h2>자식1</h2>
    <p>자식1 컨텐츠</p>
</section>

<script>
$(document).ready(function() {
    console.log('2. child1 ready');
});
</script>

<section>
    <h2>자식2</h2>
    <p>자식2 컨텐츠</p>
</section>

<script>
$(document).ready(function() {
    console.log('3. child2 ready');
});
</script>

<section>
    <h2>자식3</h2>
    <p>자식3 컨텐츠</p>
</section>

<script>
$(document).ready(function() {
    console.log('4. child3 ready');
});
</script>

</body>
</html>
```

**실행 순서**

```javascript
1. parent ready
2. child1 ready
3. child2 ready
4. child3 ready
```

---

### 1-3. JSTL Import (`<c:import>`)

**런타임 처리, 외부 URL 지원**

```javascript
<%-- ========== parent.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
    <script src="/js/jquery-3.6.0.min.js"></script>
</head>
<body>

<%-- 방법1: 직접 출력 --%>
<c:import url="header.jsp" />

<%-- 방법2: 변수에 저장 후 출력 --%>
<c:import url="footer.jsp" var="footerHtml" />
${footerHtml}

<%-- 방법3: 파라미터 전달 --%>
<c:import url="menu.jsp">
    <c:param name="activeMenu" value="home" />
</c:import>

<%-- 방법4: 외부 URL (jsp:include는 불가능) --%>
<c:import url="http://api.example.com/widget.html" />

</body>
</html>
```

---

## 2. Apache Tiles 연동

### 2-1. Tiles 설정 파일

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- ========== tiles-def.xml ========== -->
<!DOCTYPE tiles-definitions PUBLIC
    "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
    "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">

<tiles-definitions>

    <!-- 기본 레이아웃 정의 -->
    <definition name="layout.base" template="/WEB-INF/tiles/layout.jsp">
        <put-attribute name="header"  value="/WEB-INF/tiles/header.jsp" />
        <put-attribute name="menu"    value="/WEB-INF/tiles/menu.jsp" />
        <put-attribute name="content" value="" />
        <put-attribute name="footer"  value="/WEB-INF/tiles/footer.jsp" />
    </definition>

    <!-- 메인 페이지 -->
    <definition name="main" extends="layout.base">
        <put-attribute name="content" value="/WEB-INF/views/main/main.jsp" />
    </definition>

    <!-- 목록 페이지 -->
    <definition name="board.list" extends="layout.base">
        <put-attribute name="content" value="/WEB-INF/views/board/list.jsp" />
    </definition>

</tiles-definitions>
```

### 2-2. 레이아웃 및 각 컴포넌트

```javascript
<%-- ========== /WEB-INF/tiles/layout.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ taglib prefix="tiles" uri="http://tiles.apache.org/tags-tiles" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>전자정부 프레임워크</title>

    <!-- 공통 CSS -->
    <link rel="stylesheet" href="/css/common.css">

    <!-- jQuery -->
    <script src="/js/jquery-3.6.0.min.js"></script>

    <!-- 공통 JS -->
    <script src="/js/common.js"></script>

    <script>
    $(document).ready(function() {
        console.log('[1] layout.jsp ready');
    });
    </script>
</head>
<body>

    <tiles:insertAttribute name="header" />

    <div class="container">
        <tiles:insertAttribute name="menu" />
        <tiles:insertAttribute name="content" />
    </div>

    <tiles:insertAttribute name="footer" />

</body>
</html>
```

```javascript
<%-- ========== /WEB-INF/tiles/header.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<header id="header">
    <div class="logo">
        <a href="/">CHEMP 시스템</a>
    </div>
    <div class="user-info">
        <span>홍길동님</span>
        <a href="/logout">로그아웃</a>
    </div>
</header>

<script>
$(document).ready(function() {
    console.log('[2] header.jsp ready');

    // 헤더 초기화 로직
    $('#header .logo a').on('click', function(e) {
        console.log('로고 클릭');
    });
});
</script>
```

```javascript
<%-- ========== /WEB-INF/tiles/menu.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<nav id="menu">
    <ul>
        <li><a href="/board/list">게시판</a></li>
        <li><a href="/product/list">제품관리</a></li>
        <li><a href="/approval/list">승인관리</a></li>
    </ul>
</nav>

<script>
$(document).ready(function() {
    console.log('[3] menu.jsp ready');

    // 현재 페이지 메뉴 활성화
    var currentPath = window.location.pathname;
    $('#menu a').each(function() {
        if (currentPath.indexOf($(this).attr('href')) === 0) {
            $(this).addClass('active');
        }
    });
});
</script>
```

```javascript
<%-- ========== /WEB-INF/views/board/list.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<main id="content">
    <h1>게시판 목록</h1>

    <table id="boardTable">
        <thead>
            <tr>
                <th>번호</th>
                <th>제목</th>
                <th>작성자</th>
                <th>작성일</th>
            </tr>
        </thead>
        <tbody>
            <c:forEach var="item" items="${list}">
                <tr>
                    <td>${item.boardNo}</td>
                    <td>${item.title}</td>
                    <td>${item.writer}</td>
                    <td>${item.regDate}</td>
                </tr>
            </c:forEach>
        </tbody>
    </table>
</main>

<script>
$(document).ready(function() {
    console.log('[4] content(list.jsp) ready');

    // 게시판 테이블 초기화
    $('#boardTable tbody tr').on('click', function() {
        var boardNo = $(this).find('td:first').text();
        location.href = '/board/view?boardNo=' + boardNo;
    });
});
</script>
```

```javascript
<%-- ========== /WEB-INF/tiles/footer.jsp ========== --%>
<%@ page contentType="text/html; charset=UTF-8" %>
<footer id="footer">
    <p>Copyright 2024 한국화학물질관리협회</p>
</footer>

<script>
$(document).ready(function() {
    console.log('[5] footer.jsp ready');
});
</script>
```

**콘솔 출력 결과**

```javascript
[1] layout.jsp ready
[2] header.jsp ready
[3] menu.jsp ready
[4] content(list.jsp) ready
[5] footer.jsp ready
```

---

## 3. 실무 패턴

### 3-1. 전역 네임스페이스 패턴

```javascript
<%-- ========== /js/common.js 또는 layout.jsp ========== --%>
<script>
/**
 * 전역 애플리케이션 객체
 */
var APP = window.APP || {
    // 초기화 콜백 큐
    readyQueue: [],

    // 공통 데이터 저장소
    data: {},

    // 이벤트 저장소
    events: {},

    /**
     * 초기화 함수 등록
     */
    onReady: function(callback) {
        this.readyQueue.push(callback);
    },

    /**
     * 모든 초기화 함수 실행
     */
    init: function() {
        for (var i = 0; i < this.readyQueue.length; i++) {
            try {
                this.readyQueue[i]();
            } catch (e) {
                console.error('초기화 오류:', e);
            }
        }
    },

    /**
     * 커스텀 이벤트 등록
     */
    on: function(eventName, callback) {
        if (!this.events[eventName]) {
            this.events[eventName] = [];
        }
        this.events[eventName].push(callback);
    },

    /**
     * 커스텀 이벤트 발생
     */
    trigger: function(eventName, data) {
        var callbacks = this.events[eventName] || [];
        for (var i = 0; i < callbacks.length; i++) {
            callbacks[i](data);
        }
    }
};

// DOM Ready 시 모든 컴포넌트 초기화
$(document).ready(function() {
    APP.init();
});
</script>
```

```javascript
<%-- ========== header.jsp ========== --%>
<script>
APP.onReady(function() {
    console.log('header 초기화');

    // 헤더 관련 초기화 로직
});
</script>
```

```javascript
<%-- ========== content.jsp ========== --%>
<script>
APP.onReady(function() {
    console.log('content 초기화');

    // 본문 관련 초기화 로직
});
</script>
```

---

### 3-2. 비동기 의존성 처리 패턴

```javascript
<%-- ========== layout.jsp ========== --%>
<script>
$(document).ready(function() {
    // 공통 코드 데이터 로드
    $.ajax({
        url: '/api/common/codes',
        method: 'GET',
        success: function(response) {
            APP.data.codes = response;
            APP.trigger('codesLoaded', response);
        },
        error: function() {
            console.error('공통 코드 로드 실패');
        }
    });
});
</script>
```

```javascript
<%-- ========== content.jsp ========== --%>
<script>
// 공통 코드 로드 완료 후 실행
APP.on('codesLoaded', function(codes) {
    console.log('공통 코드 수신 완료');

    // 셀렉트박스 초기화
    var statusCodes = codes.STATUS || [];
    var $select = $('#statusSelect');

    statusCodes.forEach(function(code) {
        $select.append('<option value="' + code.codeId + '">' + code.codeName + '</option>');
    });
});
</script>
```

---

### 3-3. 중복 초기화 방지 패턴

```javascript
<%-- ========== common-datepicker.jsp (여러 곳에서 include됨) ========== --%>
<script>
(function() {
    // 이미 초기화되었으면 종료
    if (window._datepickerInitialized) {
        return;
    }
    window._datepickerInitialized = true;

    $(document).ready(function() {
        // 날짜 선택기 초기화 (한 번만 실행)
        $('.datepicker').datepicker({
            dateFormat: 'yy-mm-dd',
            changeMonth: true,
            changeYear: true
        });
    });
})();
</script>
```

---

### 3-4. 컴포넌트별 모듈 패턴

```javascript
<%-- ========== /WEB-INF/views/board/list.jsp ========== --%>
<script>
/**
 * 게시판 목록 모듈
 */
var BoardList = (function() {

    // private 변수
    var $table = null;
    var $searchForm = null;

    // private 함수
    function bindEvents() {
        // 행 클릭
        $table.on('click', 'tbody tr', function() {
            var boardNo = $(this).data('board-no');
            goDetail(boardNo);
        });

        // 검색 폼 제출
        $searchForm.on('submit', function(e) {
            e.preventDefault();
            search();
        });
    }

    function goDetail(boardNo) {
        location.href = '/board/view?boardNo=' + boardNo;
    }

    function search() {
        $searchForm.get(0).submit();
    }

    // public 함수
    function init() {
        $table = $('#boardTable');
        $searchForm = $('#searchForm');

        bindEvents();

        console.log('BoardList 초기화 완료');
    }

    // 외부 공개 API
    return {
        init: init
    };

})();

// DOM Ready 시 초기화
$(document).ready(function() {
    BoardList.init();
});
</script>
```

---

## 4. 비교 요약표

| 구분 | `<%@ include %>` | `<jsp:include>` | `<c:import>` | `<tiles:insertAttribute>` |
|------|-------------------|-----------------|--------------|---------------------------|
| 처리 시점 | 컴파일 타임 | 런타임 | 런타임 | 런타임 |
| 결과물 | 단일 Servlet | 별도 Servlet 호출 | 별도 요청 | RequestDispatcher |
| 파라미터 | 불가 | `<jsp:param>` | `<c:param>` | `<tiles:putAttribute>` |
| 외부 URL | 불가 | 불가 | **가능** | 불가 |
| 변수 공유 | 동일 스코프 | request 스코프 | request 스코프 | request 스코프 |
| 변경 반영 | 재컴파일 필요 | 즉시 반영 | 즉시 반영 | 즉시 반영 |
| ready 순서 | 선언 순서 | HTML 출력 순서 | HTML 출력 순서 | HTML 출력 순서 |

---

## 5. 핵심 정리

**jQuery의 ready 핸들러는 등록 순서대로 실행되며, 이는 최종 HTML에서 스크립트가 나타나는 순서와 일치합니다.**

| 상황 | document.ready 동작 |
|------|---------------------|
| `<%@ include %>` | 단일 파일로 합쳐짐, 선언 순서대로 실행 |
| `<jsp:include>` | HTML 출력 순서대로 큐에 등록, 순차 실행 |
| Tiles `insertAttribute` | `jsp:include`와 동일 |
| AJAX 동적 로딩 | DOM이 이미 ready면 즉시 실행 |

---

**출처**

- jQuery 공식 문서: [https://api.jquery.com/ready/](https://api.jquery.com/ready/)
- Apache Tiles 공식 문서: [https://tiles.apache.org/](https://tiles.apache.org/)
- JSP 2.3 Specification (JSR-245)

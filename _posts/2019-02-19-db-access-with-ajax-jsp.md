---
title: Ajax를 통한 데이터베이스 사용 - JSP
date: 2019-02-19 00:44:00 +0900
categories: [Stack, Network]
tag: [웹개발, Web, Ajax, JSP]
---

JSP 환경에서 Ajax를 통하여 데이터베이스에 접근하고, 이를 통해 간단한 데이터를 출력하는 예제입니다.

MySQL을 기준으로 작성되었습니다.

먼저 `mysql-connector-{JAVA Version}-bin.jar` 파일을 다운받아 해당 프로젝트의 `WEB-INF` 아래 `lib` 폴더에 복사합니다.

## JDBC connector 등록 및 DO 추가
프로젝트 폴더의 `src` 폴더에 패키지를 생성하고, DAO 클래스를 등록합니다.
예를들어 주소록 프로그램이라면 addrDAO.java 와 같이 적당한 이름으로 생성합니다.

```java
// JDBC connector 등록과 DO 추가
public class PortraDAO {
    Connection conn = null;
    PreparedStatement pstmt = null;
    // MySQL 연결정보
    String jdbc_driver = "com.mysql.jdbc.Driver";
    String jdbc_url = "jdbc:mysql://127.0.0.1:3306/DB명";
    // DB 연결  메서드
    void connect() {
        try {
            Class.forName(jdbc_driver);
                conn = DriverManager.getConnection(jdbc_url,"ID","PWD");;
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    // DB 연결 해제 메서드
    void disconnect() {
        if(pstmt != null) {
            try {
                pstmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

`String jdbc_url = "jdbc:mysql://127.0.0.1:3306/DB명"` 에서 DB명 부분에 사용할 데이터베이스 스키마 이름을 기입하고,

`conn = DriverManager.getConnection(jdbc_url,"ID","PWD")` 에서 ID에 해당 DB의 접근 권한이 있는 ID명, PWD에 해당 ID에 대한 패스워드를 입력합니다.

```java
// userDO 작성
import java.sql.Timestamp;
public class UserDO {
    // 멤버변수 선언
    private int us_id;
    private String us_email;
    private String us_password;
    private Timestamp us_join;
    public int getUs_id() {
        return us_id;
    }
    public void setUs_id(int us_id) {
        this.us_id = us_id;
    }
    public String getUs_email() {
        return us_email;
    }
    public void setUs_email(String us_email) {
        this.us_email = us_email;
    }
    public String getUs_password() {
        return us_password;
    }
    public void setUs_password(String us_password) {
        this.us_password = us_password;
    }
    public Timestamp getUs_join() {
        return us_join;
    }
    public void setUs_join(Timestamp us_join) {
        this.us_join = us_join;
    }
}
```

## Ajax 작성
```js
// Ajax 작성
function adminUserShowing() {
    $.ajax({
        type: "POST",
        cache: false,
        url: "ajax/adminUserShowing.jsp",
        dataType: 'xml',
        success: function(data) {
// DB 처리 성공 시 수행 내용 작성
/*
            $(data).find('user').each(function() {
                var us = "";                
                us += "<tr class='userResult'>";
                us +=     "<td class='Admin-usid'>"+$(this).find('email').text()+"</td>";
                us +=    "<td>"+$(this).find('join').text()+"</td>";
                us += "</tr>";
                
                $('.resultTable-body').append(us);
            });
*/
        },
        error: function(){
// 실패 시 수행 내용
            return false;
        }
    });
}
```

jsp 파일의 `<head>` 부분에 `<script>` 형식으로 사용해도 되고, 별도 js 파일을 만들어서 관리하여도 무관합니다.

위 코드는 관리자 페이지에서 사용자 목록을 보여주는 js 함수인데, 먼저 adminUserShowing이라는 .jsp파일을 전달합니다. 물론 처음부터 xml 파일을 생성하여 전달하여도 무관합니다.

> 단, 아랫줄의 dataType : 'xml' 이므로 파일 내용은 xml 포맷이어야 합니다.

이후 DB 처리가 성공/실패 했을 떄 동작될 내용을 작성합니다.


## JSP (XML) 파일 작성
```jsp
// jsp 형식의 xml
<?xml version="1.0" encoding="UTF-8" ?>
<%@ page import="java.util.ArrayList" %>
<%@ page import="portra.portranet.UserDO" %>
<%@ page import="portra.portranet.PortraDAO" %>
<%@ page contentType="text/xml" pageEncoding="UTF-8" %>
<%
    request.setCharacterEncoding("utf-8");
     PortraDAO user = new PortraDAO();
     ArrayList<UserDO> userList = new ArrayList<UserDO>();
     userList = user.adminUserShowing();
%>
    <contents>
<%
    for(UserDO  us : (ArrayList<UserDO>)userList) {
%>
        <user>
            <email><%=us.getUs_email() %></email>
            <join><%=us.getUs_join() %></join>
        </user>
<%
    }
%>
    </contents>
```
위에서 작성한 사용대 해당하는 DO 클래스와 데이터 처리를 담당하는 DAO 클래스를 추가하고,

`스크립틀릿`을 사용하여 DB에서 가져온 데이터를 xml 형식으로 생성합니다.

DAO 객체를 생성하고, 데이터베이스에서 회원 목록을 검색하여 ArrayList로 반환할 것이기 때문에 그것을 받을 userList를 생성합니다.

이후 DB 처리가 끝나고 `<contents>` 태그 안에서 각각의 회원을 user라는 태그로 구분한 뒤, 각 회원에 해당하는 정보를 불러와 생성합니다.


## DAO에서 DB 접근
```java
// 관리자 페이지 유저 목록 출력
public ArrayList<UserDO> adminUserShowing() {
    connect();
    ArrayList<UserDO> userList = new ArrayList<UserDO>();
    String sql = "select us_email, us_join from user_tb";
    try {
        pstmt = conn.prepareStatement(sql);
        ResultSet rs = pstmt.executeQuery();
        while(rs.next()) {
            UserDO user = new UserDO();
            user.setUs_email(rs.getString("us_email"));
            user.setUs_join(rs.getTimestamp("us_join"));
            userList.add(user);
        }
        rs.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
    finally {
        disconnect();
    }
    return userList;
}
```
자신의 DB 양식에 맞게 속성 이름과 테이블명 등을 수정하고, 아래와 같이 외부 원하는 부분에서 함수를 호출합니다.

```html
<button id="user-btn" class="admin-user-btn" onClick="adminUserShowing();">USER</button>
```


## 정리
결과적으로 DO와 DAO는 데이터베이스에서 검색 또는 삭제, 수정할 데이터들에 대한 객체를 의미합니다.

JSP나 HTML에서 버튼을 클릭하면  Ajax를 포함한 스크립트가 실행되고, 여기서 xml 파일을 데이터 형식으로 전달받는데,

이 xml 파일 안에서 DAO 함수를 실행시키고, 그 반환값을 xml 안에 데이터 형식으로 저장합니다.

마지막으로 Ajax를 사용한 스크립트에서 조회가 정상적으로 실행된 경우, find 함수를 통해 원하는 태그를 찾고, 값을 참조할 수 있습니다.

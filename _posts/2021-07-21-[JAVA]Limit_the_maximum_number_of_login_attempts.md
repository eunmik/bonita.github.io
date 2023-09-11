---
layout: post
title: "[Java] 로그인 시도 횟수 제한하기"
date: 2021-07-21
categories: [Language, Java]
tags: [Java, Login]
comments: true
---
보안을 위해서 로그인 횟수를 제한할 필요가 있다. 

로그인 횟수 제한을 위해 고려해야 할 사항은 

1. 로그인 시도 횟수를 연속 5회 이하로 제한 
2. 로그인 시도 횟수 저장 위치... 메모리 가능? DB에? 
3. 로그인 시도 횟수 초과 시 계정 잠금 (30분) 
4. 가장 최근에 로그인 실패 이후로 30분
    - 만약 계정이 잠겼는데도 시도한다면 시도했을 때부터 다시 30분 리셋

1. 로그인 데이터를 저장한 DB 테이블을 만들어준다. 

    ```sql
    CREATE TABLE `login_info` (
      `userid` varchar(128) NOT NULL,
      `count` int(10) DEFAULT '0',
      `locked_time` timestamp NULL DEFAULT NULL,
      `is_locked` tinyint(4) DEFAULT '0',
      PRIMARY KEY (`userid`),
      KEY `userid` (`userid`)
    	) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    ```

    count : 로그인 시도 횟수 

    locked_time : 로그인 시도 제한 초과 되었을 때 락 이 풀리는 시간 

    is_locked : 계정이 락 되어있는지 아닌지  1 = true, 0 = false

2. login Entity를 만들어준다. 

    ```java
    import lombok.Data;

    import javax.persistence.Column;
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.Table;
    import java.time.LocalDateTime;

    @Data
    @Entity
    @Table(name = "login_info")
    public class LoginInfo{
        
        @Id
        private String userid;
        private int count;
        @Column(name="locked_time")
        private LocalDateTime lockedTime;
        @Column(columnDefinition = "TINYINT", name="is_locked")
        private Boolean isLocked = false;

        public LoginInfo(){
        }

        public LoginInfo(String userid, int count, LocalDateTime lockedTime, Boolean isLocked){
            this.userid = userid;
            this.count = count;
            this.lockedTime = lockedTime;
            this.isLocked = isLocked;
        }

    }
    ```

3. Login 할 때 로그인 횟수/시간을 확인한다. 

    ```java
    user = userService.loadUserByUsername(username);
                lockedTime = user.getLoginCounts().getLockedTime();
                isLocked = user.getLoginCounts().getIsLocked();
                if(isLocked) {
                    if(lockedTime.isAfter(LocalDateTime.now())){ //Locked 시간이 지나지 않았으면
                        failReason = "The account is locked.";
                        lockedTime = LocalDateTime.now().plusMinutes(30);
                        throw new AccessDeniedException(failReason);
                    }else {
                        isLocked = false;
                        loginAttempts = 0;
                    }

                }
                loginAttempts = user.getLoginCounts().getCount();

                //로그인 횟수가 5번 이상이면 안된다.
                if(loginAttempts == 5 ){
                    failReason = "The maximum number of login attempts.";
                    lockedTime = LocalDateTime.now().plusMinutes(30);
                    isLocked = true;
                    throw new AccessDeniedException(failReason);
                }

    ....

    if(failReason != null ) { //로그인에 실패 했다면
    	UserLogin ulogin = new UserLogin(user.getUsername(), loginAttempts, lockedTime, isLocked);
      userLoginRepository.save(ulogin);
                    }
    ```
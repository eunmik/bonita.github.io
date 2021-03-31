---
layout: post
title: "[Java]Telegram으로 메시지 보내기"
date: 2021-03-31
excerpt: "Java에서 Telegram으로 메시지를 보내 보자!"
tags: [Java, Telegram]
comments: true
---
# Telegram에 메시지 보내기

1. Telegram에서 BotFather를 찾는다. 

<img src="https://eunmik.github.io/bonita/assets/img/210331-telegram1.png">

2.  BotFather에게 새로운 bot 생성의 도움을 요청 한다. 

```
/newbot - 새로운 bot을 생성하기 위해 
```

<img src="https://eunmik.github.io/bonita/assets/img/210331-telegram2.png">

3. 마지막 BotFather가 준 메세지에 있는 t.me/vivans_argos_bot을 누르면 새로운 대화창이 나온다. 

<img src="https://eunmik.github.io/bonita/assets/img/210331-telegram3.png">

4. Java 코드를 작성 해 준다. 

```java
public static void funcTelegram(){
        String Token = "1783392876:AAGv3A4vCQombGihQ2zf2SnnLBWHXlTwMFQ";
        String chat_id = "1384798386";
        String text = "el_abanico";

        BufferedReader in = null;
        try{
            URL obj = new URL("https://api.telegram.org/bot" + token + "/sendmessage?chat_id=" + chat_id + "&text=" + text);

            HttpURLConnection con = (HttpURLConnection) obj.openConnection();
            con.setRequestMethod("GET");
            in = new BufferedReader(new InputStreamReader(con.getInputStream(), "UTF-8"));
            String line;

            while((line = in.readLine()) != null) {
                System.out.println(line);
            }

        } catch(Exception e){
            e.printStackTrace();
        } finally {
            if(in !=null) try { in.close();} catch(Exception e) { e.printStackTrace();}
        }

    }
```
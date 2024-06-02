---
title: Springboot读取邮箱邮件
date: 2024-06-02 23:45:02
tags:
---

邮件扫描接口
```java
package com.example.wallhaven.biz.mail;

/**
 * @author ALVIN
 */
public interface MailReceiptService {

    /**
     * 扫描读取邮箱内邮件
     */
    void scan();
}

```

邮件处理接口
```java
package com.example.wallhaven.biz.mail;

import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.search.SearchTerm;

/**
 * @author ALVIN
 */
public interface MailHandler {

    /**
     * 邮件筛选器
     * @return 邮件筛选器
     */
    SearchTerm mailFilter();

    /**
     * 邮件消息处理
     * @param message 邮件
     */
    void handler(Message message);
}

```

POP3协议邮件扫描实现类
```java
package com.example.wallhaven.biz.mail;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import javax.mail.*;
import javax.mail.search.SearchTerm;
import java.util.List;
import java.util.Optional;
import java.util.Properties;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * @author ALVIN
 */
@Slf4j
@RequiredArgsConstructor
public class Pop3MailReceiptServiceImpl implements MailReceiptService {

    /**
     * 协议
     */
    private static final String PROTOCOL = "pop3";
    /**
     * 端口
     */
    private static final String PORT = "995";

    private final String hostname;

    private final String username;

    private final String password;

    private final MailHandler mailHandler;

    @Override
    public void scan() {
        try {
            // 1. 设置连接信息, 生成一个 Session
            // 获取连接
            Session session;
            session = Session.getInstance(getPop3());
            session.setDebug(false);
            // 2. 获取Store, 并连接到服务器
            Store store = session.getStore(PROTOCOL);

            // POP3服务器的登陆认证
            store.connect(hostname, username, password);
            // 默认父目录
            Folder defaultFolder = store.getDefaultFolder();
            if (defaultFolder == null) {
                log.error("服务器不可用！ hostServer:{}", hostname);
                return;
            }
            // 获取收件箱
            Folder folder = defaultFolder.getFolder("INBOX");
            // 可读邮件,可以删邮件的模式打开目录
            folder.open(Folder.READ_WRITE);
            // 取出来邮件数
            log.info("folder urlName: {}, 共有邮件: {}封, UnreadMessages: {}, NewMessages: {}...", folder.getURLName().toString(),
                    folder.getMessageCount(), folder.getUnreadMessageCount(), folder.getNewMessageCount());

            List<Message> mails = searchMails(folder);
            mails.forEach(mailHandler::handler);


            // 7. 关闭 Folder 会真正删除邮件, false 不删除
            folder.close(false);
            // 8. 关闭 store, 断开网络连接
            store.close();

        } catch (MessagingException e) {
            log.error(e.getMessage(), e);
        }
    }

    /**
     * 获取POP3收信配置 995
     * @return 邮箱POP3连接配置属性
     */
    private Properties getPop3() {
        Properties props = new Properties();
        props.setProperty("mail.transport.protocol", PROTOCOL);
        // 按需要更改
        props.setProperty("mail.pop3.host", hostname);
        props.setProperty("mail.pop3.port", PORT);
        // SSL安全连接参数
        props.setProperty("mail.pop3.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
        props.setProperty("mail.pop3.socketFactory.fallback", "false");
        props.setProperty("mail.pop3.socketFactory.port", PORT);
        // 解决DecodingException: BASE64Decoder: but only got 0 before padding character (=)
        props.setProperty("mail.mime.base64.ignore-errors", "true");
        return props;
    }

    private List<Message> searchMails(Folder folder) throws MessagingException {
        //根据设置好的条件获取message
        SearchTerm searchTerm = mailHandler.mailFilter();
        Message[] messages = searchTerm == null
                ? folder.getMessages()
                : folder.search(searchTerm);

        log.info("search邮件: " + messages.length + "封");
        return Stream.of(messages).collect(Collectors.toList());
    }
}

```

邮件处理器实现类
```java
package com.example.wallhaven.biz.mail;

import com.example.wallhaven.exception.BaseException;
import lombok.extern.slf4j.Slf4j;

import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.search.*;
import java.util.Date;
import java.util.concurrent.TimeUnit;

/**
 * @author ALVIN
 */
@Slf4j
public class SimpleMailHandler implements MailHandler{
    @Override
    public SearchTerm mailFilter() {
        // 邮件主题带有“神策报表”
        final String subjectKeyword = "神策报表";
        SubjectTerm subjectTerm = new SubjectTerm(subjectKeyword);
        // 发送时间在24小时之内
        long oneDaysAgo = System.currentTimeMillis() - TimeUnit.HOURS.toMillis(24);
        SentDateTerm receivedDateTerm = new SentDateTerm(ComparisonTerm.GT, new Date(oneDaysAgo));
        // 来自指定的发件人
        final String fromEmail = "543046534@qq.com";
        FromStringTerm fromStringTerm = new FromStringTerm(fromEmail);
        return new AndTerm(new SearchTerm[]{subjectTerm, receivedDateTerm, fromStringTerm});
    }

    @Override
    public void handler(Message message) {
        try {
            String subject = message.getSubject();
            log.info("收到邮件：{}", subject);
        } catch (MessagingException e) {
            throw new BaseException("邮件处理失败", e);
        }
    }
}
```

使用demo
```java
package com.example.wallhaven;

import com.example.wallhaven.biz.mail.MailHandler;
import com.example.wallhaven.biz.mail.MailReceiptService;
import com.example.wallhaven.biz.mail.Pop3MailReceiptServiceImpl;
import com.example.wallhaven.biz.mail.SimpleMailHandler;
import jodd.util.CommandLine;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.EnableScheduling;

/**
 * @author Alvin
 */
@MapperScan("com.example.wallhaven.mapper")
@EnableAsync
@EnableScheduling
@SpringBootApplication
@MapperScan("com.example.wallhaven.mapper")
public class WallhavenApplication {



    public static void main(String[] args) {
        SpringApplication.run(WallhavenApplication.class, args);
    }

    public void run(String... args) throws Exception {
        String hostname = "pophz.qiye.163.com";
        String username = "xxxxxx@qq.com";
        String password = "xxxxxxx";
        MailHandler mailHandler = new SimpleMailHandler();
        MailReceiptService mailReceiptService = new Pop3MailReceiptServiceImpl(hostname, username, password, mailHandler);
        mailReceiptService.scan();

    }
}

```

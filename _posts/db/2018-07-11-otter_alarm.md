---
layout: post
title: "otter支持钉钉短信等报警"
keywords: ["otter"]
description: "otter"
category: "MySQL"
tags: ["otter","binlog"]
---

目前只支持邮件报警，发送邮件的代码在DefaultAlarmService里边

```
public void doSend(AlarmMessage data) throws Exception {
    SimpleMailMessage mail = new SimpleMailMessage(); // 只发送纯文本
    mail.setFrom(username);
    mail.setSubject(TITLE);// 主题
    mail.setText(data.getMessage());// 邮件内容
    String receiveKeys[] = StringUtils.split(StringUtils.replace(data.getReceiveKey(), ";", ","), ",");

    SystemParameter systemParameter = systemParameterService.find();
    List<String> mailAddress = new ArrayList<String>();
    for (String receiveKey : receiveKeys) {
        String receiver = convertToReceiver(systemParameter, receiveKey);
        String strs[] = StringUtils.split(StringUtils.replace(receiver, ";", ","), ",");
        for (String str : strs) {
            if (isMail(str)) {
                if (str != null) {
                    mailAddress.add(str);
                }
            } else if (isSms(str)) {
                // do nothing
            }
        }
    }

    if (!mailAddress.isEmpty()) {
        mail.setTo(mailAddress.toArray(new String[mailAddress.size()]));
        doSendMail(mail);
    }
}
```

首先会从报警规则里的receiveKey，获取receiver，这个receiver应该是按";"分割的，这里先替换“;”为“,”，然后按照","分割

所以只要增加钉钉，手机号码的代码就好了

```
public void doSend(AlarmMessage data) throws Exception {
    SimpleMailMessage mail = new SimpleMailMessage(); // 只发送纯文本
    mail.setFrom(username);
    mail.setSubject(TITLE);// 主题
    mail.setText(data.getMessage());// 邮件内容
    String receiveKeys[] = StringUtils.split(StringUtils.replace(data.getReceiveKey(), ";", ","), ",");

    SystemParameter systemParameter = systemParameterService.find();
    List<String> mailAddress = new ArrayList<String>();
    for (String receiveKey : receiveKeys) {
        String receiver = convertToReceiver(systemParameter, receiveKey);
        String strs[] = StringUtils.split(StringUtils.replace(receiver, ";", ","), ",");
        for (String str : strs) {
            if (isMail(str)) {
                if (str != null) {
                    mailAddress.add(str);
                }
            } else if (isSms(str)) {
                // do nothing
            }
        }
    }

    if (!mailAddress.isEmpty()) {
        mail.setTo(mailAddress.toArray(new String[mailAddress.size()]));
        doSendMail(mail);
    }
    doSendDDmsg(String.format("from otter alarm msg: %s",data.getMessage()));
}
```

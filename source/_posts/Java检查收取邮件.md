---
title: Java检查收取邮件
top: false
cover: false
password: ''
toc: true
mathjax: true
summary: JMail收取邮件，采用网上找的demo，并对其中的邮件格式有误的地址添加解析方法。
tags:
  - JMail
categories:
  - Java
date: 2019-08-31 16:17:52
---

```java
import org.apache.commons.lang3.StringUtils;

import javax.mail.*;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeUtility;
import java.io.*;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Properties;

/**
 * 检查收取邮件
 *
 * @author: 张勇波
 */
public class CheckEmail {
    private MimeMessage mimeMessage = null;
    private String saveAttachPath = ""; //附件下载后的存放目录,默认：d:\tmp 或 /tem
    private StringBuffer bodytext = new StringBuffer();//邮件内容
    private String dateformat = "yyyy-MM-dd HH:mm"; //默认日期显示格式

    public static void main(String args[]) throws Exception {
        String host = "pop.exmail.qq.com";
        String port = "110";

        String email = "****@bis.com.cn";
        String emailPWD = "*******";
        Message message[] = CheckEmail.checkEmail(host, port, email, emailPWD);
        System.out.println("Messages's length: " + message.length);
        CheckEmail ce;
        for (int i = 0; i < message.length; i++) {
            System.out.println("--------------------------------------------------------");
            ce = new CheckEmail((MimeMessage) message[i]);
            if (ce.isNew()) {
                continue;
            } else {
//                ce.setDateFormat("yyyy年MM月dd日 HH:mm");
                System.out.println("Message " + i + " 邮件主题: " + ce.getSubject());
                System.out.println("Message " + i + " 邮件发送日期: " + ce.getSentDate());
                System.out.println("Message " + i + " 是否需要回执: " + ce.getReplySign());
//                System.out.println("Message " + i + " 是否已读: " + ce.isNew());  //pop3不支持若是已读 则不处理后续
                System.out.println("Message " + i + " 是否包含附件: " + ce.isContainAttach(message[i]));
                System.out.println("Message " + i + " 发件人: " + ce.getFrom());
                System.out.println("Message " + i + " 收件人: " + ce.getMailAddress("to"));
                System.out.println("Message " + i + " 抄送: " + ce.getMailAddress("cc"));
                System.out.println("Message " + i + " 密送: " + ce.getMailAddress("bcc"));
                System.out.println("Message " + i + " Message-ID: " + ce.getMessageId());
                System.out.println("Message " + i + " MessageNumber: " + message[i].getMessageNumber());
//                ce.getMailContent(message[i]);// 获得邮件正文
//                System.out.println("Message " + i + " 正文: \r\n" + ce.getBodyText());
//                ce.setAttachPath("d:\\");//设置附件保存路径
//                ce.saveAttachMent(message[i]);//保存附件
//                if (i == 10) break;
//                message[i].setFlag(Flags.Flag.SEEN, true);//设置为已读，pop3不能使用。。。。
//                message[i].saveChanges();
            }
        }
    }

    /**
     * 获取邮件信息
     *
     * @param host     mail.pop3.port
     * @param port     mail.pop3.port
     * @param email    邮箱地址
     * @param emailPWD 邮箱密码
     * @return Message[] 邮件信息数组
     * 如需获取邮件正文，需要getMailContent(message[i])后getBodyText()
     * 如需保存附件 saveAttachMent(message[i])
     * @throws MessagingException 收取邮件错误
     */
    public static Message[] checkEmail(String host, String port, String email, String emailPWD) throws MessagingException {
        Properties props = new Properties();
        props.setProperty("mail.store.protocol", "pop3");
        props.setProperty("mail.pop3.port", port);
        props.setProperty("mail.pop3.host", host);
//        props.setProperty("mail.store.protocol", "imap");
//        props.setProperty("mail.imap.port", port);
//        props.setProperty("mail.imap.host", host);
//        props.setProperty("mail.imap.auth", "true");
        Session session = Session.getInstance(props);
        Store store = session.getStore("pop3");
        store.connect(email, emailPWD);
        Folder folder = store.getFolder("INBOX");
        folder.open(Folder.READ_WRITE);
        System.setProperty("mail.mime.multipart.ignoreexistingboundaryparameter", "true");
        System.out.println("-----getUnreadMessageCount----" + folder.getMessageCount());
        Message message[] = folder.getMessages();
        return message;
    }

    public CheckEmail(MimeMessage mimeMessage) {
        this.mimeMessage = mimeMessage;
    }

    /**
     * 设置附件存放路径
     *
     * @param attachpath 附件路径
     */
    public void setAttachPath(String attachpath) {
        this.saveAttachPath = attachpath;
    }

    /**
     * 设置日期显示格式
     *
     * @param format 如yyyy-MM-dd HH:mm
     */
    public void setDateFormat(String format) {
        this.dateformat = format;
    }

    /**
     * 获得附件存放路径
     *
     * @return 附件存放路径
     */
    public String getAttachPath() {
        return saveAttachPath;
    }

    /**
     * 获得发件人的地址和姓名
     *
     * @return 名称＜地址＞
     * @throws MessagingException 获取发件人错误
     */
    public String getFrom() throws MessagingException {
        InternetAddress address[] = (InternetAddress[]) mimeMessage.getFrom();
        String from = address[0].getAddress();
        if (from == null)
            from = "";
        String personal = address[0].getPersonal();
        if (personal == null)
            personal = "";
        String fromaddr = personal + "<" + from + ">";
        return fromaddr;
    }

    /**
     * 对格式有误的邮件地址进行解析处理
     *
     * @param num 地址数组
     * @return
     */
    private InternetAddress[] getAddress(String[] num) {
        InternetAddress[] address;
        List<InternetAddress> addresses = new ArrayList<InternetAddress>();
        for (String s : num) {
            String[] to2 = s.split(",");
            for (String s2 : to2) {
                InternetAddress internetAddress = new InternetAddress();
                String[] mail = s2.split("<");
                if (StringUtils.isNotBlank(mail[0])) {
                    try {
                        internetAddress.setPersonal(mail[0]);
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                    }
                } else {
                    try {
                        internetAddress.setPersonal("");
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                    }
                }
                internetAddress.setAddress(mail[1]);
                addresses.add(internetAddress);
            }
        }
        InternetAddress[] ia = new InternetAddress[addresses.size()];
        address = addresses.toArray(ia);
        return address;
    }

    /**
     * 获得邮件的收件人，抄送，和密送的地址和姓名，根据所传递的参数的不同
     *
     * @param type 接收方类型
     *             "to"----收件人
     *             "cc"---抄送人地址
     *             "bcc"---密送人地址
     * @return 名称＜地址＞
     * @throws Exception 获取收件人错误
     */
    public String getMailAddress(String type) throws Exception {
        StringBuilder mailaddr = new StringBuilder();
        String addtype = type.toUpperCase();
        InternetAddress[] address;
        if (addtype.equals("TO") || addtype.equals("CC") || addtype.equals("BCC")) {
            if (addtype.equals("TO")) {
                try {
                    address = (InternetAddress[]) mimeMessage.getRecipients(Message.RecipientType.TO);
                } catch (MessagingException e) {
                    String[] to = mimeMessage.getHeader(Message.RecipientType.TO.toString());
                    address = getAddress(to);
                }
            } else if (addtype.equals("CC")) {
                try {
                    address = (InternetAddress[]) mimeMessage.getRecipients(Message.RecipientType.CC);
                } catch (MessagingException e) {
                    String[] cc = mimeMessage.getHeader(Message.RecipientType.CC.toString());
                    address = getAddress(cc);
                }
            } else {
                try {
                    address = (InternetAddress[]) mimeMessage.getRecipients(Message.RecipientType.BCC);
                } catch (MessagingException e) {
                    String[] bcc = mimeMessage.getHeader(Message.RecipientType.BCC.toString());
                    address = getAddress(bcc);
                }
            }
            if (address != null) {
                for (int i = 0; i < address.length; i++) {
                    String email = address[i].getAddress();
                    if (email == null)
                        email = "";
                    else {
                        email = MimeUtility.decodeText(email).replaceAll(">>", "").replaceAll(">", "").replaceAll("\r\n\t", "").trim();
                    }
                    String personal = address[i].getPersonal();
                    if (personal == null)
                        personal = "";
                    else {
                        personal = MimeUtility.decodeText(personal).replaceAll("&#39;", "").replaceAll("'", "").replaceAll("\"", "").trim();
                    }
                    String compositeto = personal + "<" + email + ">";
                    mailaddr.append(",").append(compositeto);
                }
                mailaddr = new StringBuilder(mailaddr.substring(1));
            }
        } else {
            throw new Exception("getMailAddress 没有此类型收件人!");
        }
        return mailaddr.toString();
    }

    /**
     * 获得邮件主题
     *
     * @return 邮件主题
     * @throws MessagingException           获取主题错误
     * @throws UnsupportedEncodingException 转换文本错误
     */
    public String getSubject() throws MessagingException, UnsupportedEncodingException {
        String subject = MimeUtility.decodeText(mimeMessage.getSubject());
        return subject == null ? "" : subject;
    }

    /**
     * 获得邮件发送日期
     *
     * @return 邮件发送日期
     * @throws MessagingException 获取邮件发送日期错误
     */
    public String getSentDate() throws MessagingException {
        Date sentdate = mimeMessage.getSentDate();
        SimpleDateFormat format = new SimpleDateFormat(dateformat);
        return format.format(sentdate);
    }

    /**
     * 获得邮件正文内容
     *
     * @return 邮件正文内容
     */
    public String getBodyText() {
        return bodytext.toString();
    }

    /**
     * 解析邮件，把得到的邮件内容保存到一个StringBuffer对象中，
     * 解析邮件 主要是根据MimeType类型的不同执行不同的操作，一步一步的解析
     *
     * @param part 附件对象
     * @throws Exception 邮件解析错误
     */
    public void getMailContent(Part part) throws Exception {
        String contenttype = part.getContentType();
        int nameindex = contenttype.indexOf("name");
        boolean conname = false;
        if (nameindex != -1)
            conname = true;
        System.out.println("--getMailContent--ContentType: " + contenttype);
        if (part.isMimeType("text/plain") && !conname) {
            bodytext.append((String) part.getContent());
        } else if (part.isMimeType("text/html") && !conname) {
            bodytext.append((String) part.getContent());
        } else if (part.isMimeType("multipart/*")) {
            Multipart multipart = (Multipart) part.getContent();
            int counts = multipart.getCount();
            for (int i = 0; i < counts; i++) {
                getMailContent(multipart.getBodyPart(i));
            }
        } else if (part.isMimeType("message/rfc822")) {
            getMailContent((Part) part.getContent());
        }
    }

    /**
     * 判断此邮件是否需要回执，如果需要回执返回"true",否则返回"false"
     *
     * @return 是否需要回执
     * @throws MessagingException 邮件获取错误
     */
    public boolean getReplySign() throws MessagingException {
        boolean replysign = false;
        String needreply[] = mimeMessage.getHeader("Disposition-Notification-To");
        if (needreply != null) {
            replysign = true;
        }
        return replysign;
    }

    /**
     * 获得此邮件的Message-ID
     *
     * @return Message-ID
     * @throws MessagingException 邮件获取错误
     */
    public String getMessageId() throws MessagingException {
        return mimeMessage.getMessageID();
    }

    /**
     * 判断此邮件是否已读，如果未读返回返回false,反之返回true
     *
     * @return 是否已读
     * @throws MessagingException 邮件获取错误
     */
    public boolean isNew() throws MessagingException {
        boolean isnew = false;
        Flags flags = (mimeMessage).getFlags();
        Flags.Flag[] flag = flags.getSystemFlags();
        for (int i = 0; i < flag.length; i++) {
            if (flag[i] == Flags.Flag.SEEN) {
                isnew = true;
                break;
            }
        }
        return isnew;
    }

    /**
     * 判断此邮件是否包含附件
     *
     * @param part 附件对象
     * @return 是否包含附件
     * @throws Exception 邮件附件判断错误
     */
    public boolean isContainAttach(Part part) throws Exception {
        boolean attachflag = false;
        if (part.isMimeType("multipart/*")) {
            Multipart mp = (Multipart) part.getContent();
            for (int i = 0; i < mp.getCount(); i++) {
                BodyPart mpart = mp.getBodyPart(i);
                String disposition = mpart.getDisposition();
                if ((disposition != null)
                        && ((disposition.equals(Part.ATTACHMENT)) || (disposition
                        .equals(Part.INLINE))))
                    attachflag = true;
                else if (mpart.isMimeType("multipart/*")) {
                    attachflag = isContainAttach(mpart);
                } else {
                    String contype = mpart.getContentType();
                    if (contype.toLowerCase().indexOf("application") != -1)
                        attachflag = true;
                    if (contype.toLowerCase().indexOf("name") != -1)
                        attachflag = true;
                }
            }
        } else if (part.isMimeType("message/rfc822")) {
            attachflag = isContainAttach((Part) part.getContent());
        }
        return attachflag;
    }

    /**
     * 保存附件
     *
     * @param part 附件对象
     * @throws Exception 邮件保存错误
     */
    public void saveAttachMent(Part part) throws Exception {
        String fileName;
        if (part.isMimeType("multipart/*")) {
            Multipart mp = (Multipart) part.getContent();
            for (int i = 0; i < mp.getCount(); i++) {
                BodyPart mpart = mp.getBodyPart(i);
                String disposition = mpart.getDisposition();
                if ((disposition != null)
                        && ((disposition.equals(Part.ATTACHMENT))
                        || (disposition.equals(Part.INLINE)))) {
                    fileName = mpart.getFileName();
                    if (fileName == null) continue;
//                    if (fileName.toLowerCase().indexOf("gb2312") != -1) {
                    fileName = MimeUtility.decodeText(fileName);
//                    }
                    saveFile(fileName, mpart.getInputStream());
                } else if (mpart.isMimeType("multipart/*")) {
                    saveAttachMent(mpart);
                } else {
                    fileName = mpart.getFileName();
                    if ((fileName != null)
                            && ((fileName.toLowerCase().indexOf("GB2312") != -1) || (fileName.toLowerCase().indexOf("gb18030") != -1))) {
                        fileName = MimeUtility.decodeText(fileName);
                        saveFile(fileName, mpart.getInputStream());
                    }
                }
            }
        } else if (part.isMimeType("message/rfc822")) {
            saveAttachMent((Part) part.getContent());
        }
    }

    /**
     * 真正的保存附件到指定目录里
     *
     * @param fileName 文件名
     * @param in       附件InputStream对象
     * @throws Exception 附件保存错误
     */
    private void saveFile(String fileName, InputStream in) throws Exception {
        String osName = System.getProperty("os.name");
        String storedir = getAttachPath();
        if (osName == null)
            osName = "";
        if (osName.toLowerCase().contains("win")) {
            if (storedir == null || storedir.equals(""))
                storedir = "d:\\tmp";
        } else {
            storedir = "/tmp";
        }
        File storefile = new File(storedir + System.getProperty("file.separator") + fileName);
        System.out.println("--saveFile--附件保存路径: " + storefile.toString());
        BufferedOutputStream bos = null;
        BufferedInputStream bis = null;
        try {
            bos = new BufferedOutputStream(new FileOutputStream(storefile));
            bis = new BufferedInputStream(in);
            int c;
            while ((c = bis.read()) != -1) {
                bos.write(c);
                bos.flush();
            }
        } catch (Exception exception) {
            exception.printStackTrace();
            throw new Exception("文件保存失败!");
        } finally {
            assert bos != null;
            bos.close();
            assert bis != null;
            bis.close();
        }
    }
}


```
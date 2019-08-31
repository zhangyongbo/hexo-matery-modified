---
title: JAVA后台调用http接口
top: false
cover: false
password: ''
toc: true
mathjax: true
summary: Java后台使用HttpURLConnection进行http请求调用
tags:
  - Java
  - Http
categories:
  - Java
date: 2019-08-31 16:13:19
---

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLEncoder;
import java.util.Iterator;
import java.util.Map;

/**
 * 调用http接口
 * Created by 张勇波 on 2017/12/12.
 */
public class HttpURL {
    /**
     * 调用http接口
     *
     * @param strPostURL  url地址
     * @param map         参数集合
     * @param method      GET, POST, HEAD, OPTIONS, PUT, DELETE, TRACE
     * @param contentType 请求类型
     *                    "application/x-javascript" json数据
     *                    "text/xml" xml数据
     *                    "application/x-www-form-urlencoded" 表单数据 浏览器默认类型
     */
    public static String httpURLConnection(String strPostURL, Map<String, String> map, String contentType, String method) {
        contentType =
                contentType != null && contentType.length() > 0 ? contentType : "application/x-www-form-urlencoded";
        try {
            method = method != null ? method : "POST";
            URL url = new URL(strPostURL);
            // 将url 以 open方法返回的urlConnection  连接强转为HttpURLConnection连接  (标识一个url所引用的远程对象连接)
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();// 此时cnnection只是为一个连接对象,待连接中
            // 设置连接输出流为true,默认false (post 请求是以流的方式隐式的传递参数)
            connection.setDoOutput(true);
            // 设置连接输入流为true
            connection.setDoInput(true);
            // 设置请求方式
            connection.setRequestMethod(method.toUpperCase());
            // post请求缓存设为false
            connection.setUseCaches(false);
            // 设置该HttpURLConnection实例是否自动执行重定向
            connection.setInstanceFollowRedirects(true);
            // 设置请求头里面的各个属性 (以下为设置内容的类型,设置为经过urlEncoded编码过的from参数)
            connection.setRequestProperty("accept", "*/*");
            connection.setRequestProperty("connection", "Keep-Alive");
            connection.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            connection.setRequestProperty("Content-Type", contentType + ";charset=utf-8");
            connection.setRequestProperty("charset", "utf-8");
            StringBuilder sb = new StringBuilder();
            if (map != null && map.size() > 0) {
                Iterator iter = map.entrySet().iterator();
                while (iter.hasNext()) {
                    Map.Entry entry = (Map.Entry) iter.next();
                    Object key = entry.getKey();
                    Object val = entry.getValue();
                    if (sb.length() == 0) {
                        sb.append(key + "=" + URLEncoder.encode((String) val, "utf-8"));
                    } else {
                        sb.append("&" + key + "=" + URLEncoder.encode((String) val, "utf-8"));
                    }
                }
                connection.setRequestProperty("Content-length", String.valueOf(sb.toString().length()));
                // 创建输入输出流,用于往连接里面输出携带的参数,(输出内容为?后面的内容)
                PrintWriter printWriter = new PrintWriter(connection.getOutputStream());
                // 将参数输出到连接
                printWriter.write(sb.toString());
                // 输出完成后刷新并关闭流
                printWriter.flush();
                // 重要且易忽略步骤 (关闭流,切记!)
                printWriter.close();
            }
            // 建立连接 (请求未开始,直到connection.getInputStream()方法调用时才发起,以上各个参数设置需在此方法之前进行)
            connection.connect();
            // 连接发起请求,处理服务器响应  (从连接获取到输入流并包装为bufferedReader)
            BufferedReader bf = new BufferedReader(new InputStreamReader(connection.getInputStream(), "UTF-8"));
            String line;
            StringBuilder result = new StringBuilder(); // 用来存储响应数据
            // 循环读取流,若不到结尾处
            while ((line = bf.readLine()) != null) {
                result.append(line).append(System.getProperty("line.separator"));
            }
            bf.close();// 重要且易忽略步骤 (关闭流,切记!)
            connection.disconnect(); // 销毁连接
            return result.toString();
        } catch (Exception e) {
            e.printStackTrace();
            return "httpURLConnection调用失败";
        }
    }
}
```
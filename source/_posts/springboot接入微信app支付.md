---
title: springboot接入微信app支付
comments: false
date: 2019-08-02 21:20:43
categories: 
 - springBoot
 - 微信支付
tags: 
 - springBoot
 - 微信支付
---

### 一：集成步骤

#### 1.引入依赖:

```java
<dependency>
  <groupId>com.github.wxpay</groupId>
  <artifactId>wxpay-sdk</artifactId>
  <version>0.0.3</version>
</dependency>
```

#### 2.微信app支付参数配置:

```properties
#服务器域名地址
server.service-domain = http://127.0.0.1:8080

#微信app支付
pay.wxpay.app.appID = "你的appid"
pay.wxpay.app.mchID = "你的商户id"
pay.wxpay.app.key = "你的api秘钥，不是appSecret"
#从微信商户平台下载的安全证书存放的路径、我放在resources下面,切记一定要看看target目录下的class文件下有没有打包apiclient_cert.p12文件
pay.wxpay.app.certPath = static/cert/wxpay/apiclient_cert.p12
#微信支付成功的异步通知接口
pay.wxpay.app.payNotifyUrl=${server.service-domain}/api/wxPay/notify
```

#### 3.定义配置类:

```java
package com.annaru.upms.payment.config;

import com.github.wxpay.sdk.WXPayConfig;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.io.InputStream;

/**
 * 配置我们自己的信息
 */
@Component
@ConfigurationProperties(prefix = "pay.wxpay.app")
public class WxPayAppConfig implements WXPayConfig {
    /**
     * appID
     */
    private String appID;

    /**
     * 商户号
     */
    private String mchID;

    /**
     * API 密钥
     */
    private String key;

    /**
     * API证书绝对路径 (本项目放在了 resources/cert/wxpay/apiclient_cert.p12")
     */
    private String certPath;

    /**
     * HTTP(S) 连接超时时间，单位毫秒
     */
    private int httpConnectTimeoutMs = 8000;

    /**
     * HTTP(S) 读数据超时时间，单位毫秒
     */
    private int httpReadTimeoutMs = 10000;

    /**
     * 微信支付异步通知地址
     */
    private String payNotifyUrl;

    /**
     * 微信退款异步通知地址
     */
    private String refundNotifyUrl;

    /**
     * 获取商户证书内容（这里证书需要到微信商户平台进行下载）
     *
     * @return 商户证书内容
     */
    @Override
    public InputStream getCertStream() {
        InputStream certStream  =getClass().getClassLoader().getResourceAsStream(certPath);
        return certStream;
    }

    public String getAppID() {
        return appID;
    }

    public void setAppID(String appID) {
        this.appID = appID;
    }

    public String getMchID() {
        return mchID;
    }

    public void setMchID(String mchID) {
        this.mchID = mchID;
    }

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public String getCertPath() {
        return certPath;
    }

    public void setCertPath(String certPath) {
        this.certPath = certPath;
    }

    public int getHttpConnectTimeoutMs() {
        return httpConnectTimeoutMs;
    }

    public void setHttpConnectTimeoutMs(int httpConnectTimeoutMs) {
        this.httpConnectTimeoutMs = httpConnectTimeoutMs;
    }

    public int getHttpReadTimeoutMs() {
        return httpReadTimeoutMs;
    }

    public void setHttpReadTimeoutMs(int httpReadTimeoutMs) {
        this.httpReadTimeoutMs = httpReadTimeoutMs;
    }

    public String getPayNotifyUrl() {
        return payNotifyUrl;
    }

    public void setPayNotifyUrl(String payNotifyUrl) {
        this.payNotifyUrl = payNotifyUrl;
    }

    public String getRefundNotifyUrl() {
        return refundNotifyUrl;
    }

    public void setRefundNotifyUrl(String refundNotifyUrl) {
        this.refundNotifyUrl = refundNotifyUrl;
    }
}
```

#### 4. 定义controller:

> 在调用微信服务接口进行统一下单之前，
>
> 1、为保证安全性，建议验证数据库是否存在订单号对应的订单。

```java
package com.annaru.upms.payment.controller;

import com.annaru.common.result.ResultMap;
import com.annaru.upms.payment.service.WxPayService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;

@Api(tags = "微信支付接口管理")
@RestController
@RequestMapping("/wxPay")
public class WxPayController{

    @Autowired
    private WxPayService wxPayService;

    /**
     * 统一下单接口
     */
    @ApiOperation(value = "统一下单", notes = "统一下单")
    @GetMapping("/unifiedOrder")
    public ResultMap unifiedOrder(
        @ApiParam(value = "订单号") @RequestParam String orderNo,
        @ApiParam(value = "订单金额") @RequestParam double amount,
        @ApiParam(value = "商品名称") @RequestParam String body,
                                  HttpServletRequest request) {
        try {
            // 1、验证订单是否存在
            
            // 2、开始微信支付统一下单
            ResultMap resultMap = wxPayService.unifiedOrder(orderNo, orderNo, body);
            return resultMap;//系统通用的返回结果集，见文章末尾
        } catch (Exception e) {
            logger.error(e.getMessage());
            return ResultMap.error("运行异常，请联系管理员");
        }
    }
    
    /**
     * 微信支付异步通知
     */
    @RequestMapping(value = "/notify")
    public String payNotify(HttpServletRequest request) {
        InputStream is = null;
        String xmlBack = "<xml><return_code><![CDATA[FAIL]]></return_code><return_msg><![CDATA[报文为空]]></return_msg></xml> ";
        try {
            is = request.getInputStream();
            // 将InputStream转换成String
            BufferedReader reader = new BufferedReader(new InputStreamReader(is));
            StringBuilder sb = new StringBuilder();
            String line = null;
            while ((line = reader.readLine()) != null) {
                sb.append(line + "\n");
            }
            xmlBack = wxPayService.notify(sb.toString());
        } catch (Exception e) {
            logger.error("微信手机支付回调通知失败：", e);
        } finally {
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return xmlBack;
    }
    
    @ApiOperation(value = "退款", notes = "退款")
    @PostMapping("/refund")
    public ResultMap refund(@ApiParam(value = "订单号") @RequestParam String orderNo,
                            @ApiParam(value = "退款金额") @RequestParam double amount,
                            @ApiParam(value = "退款原因") @RequestParam(required = false) String refundReason){

        return wxPayService.refund(orderNo, amount, refundReason);
    }

}
```

#### 5、定义service接口：

```java
package com.annaru.upms.payment.service;

import com.annaru.common.result.ResultMap;

/**
 * 微信支付服务接口
 */
public interface WxPayService {

    /**
     * @Description: 微信支付统一下单
     * @param orderNo: 订单编号
     * @param amount: 实际支付金额
     * @param body: 订单描述
     * @Author: 
     * @Date: 2019/8/1
     * @return
     */
    ResultMap unifiedOrder(String orderNo, double amount, String body) ;

    /**
     * @Description: 订单支付异步通知
     * @param notifyStr: 微信异步通知消息字符串
     * @Author: 
     * @Date: 2019/8/1
     * @return 
     */
    String notify(String notifyStr) throws Exception;
    
    /**
     * @Description: 退款
     * @param orderNo: 订单编号
     * @param amount: 实际支付金额
     * @param refundReason: 退款原因
     * @Author: XCK
     * @Date: 2019/8/6
     * @return
     */
    ResultMap refund(String orderNo, double amount, String refundReason);

}

```

#### 6、service实现类

```java
package com.annaru.upms.payment.service.impl;

import com.alibaba.dubbo.config.annotation.Reference;
import com.annaru.common.result.ResultMap;
import com.annaru.common.util.HttpContextUtils;
import com.annaru.upms.payment.config.WxPayAppConfig;
import com.annaru.upms.payment.service.WxPayService;
import com.annaru.upms.service.IOrderPaymentService;
import com.github.wxpay.sdk.WXPay;
import com.github.wxpay.sdk.WXPayUtil;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Service
public class WxPayServiceImpl implements WxPayService {
    private final Logger logger = LoggerFactory.getLogger(WxPayServiceImpl.class);

    @Reference
    private IOrderPaymentService orderPaymentService;
    @Autowired
    private WxPayAppConfig wxPayAppConfig;

    @Override
    public ResultMap unifiedOrder(String orderNo, double amount, String body) {
        Map<String, String> returnMap = new HashMap<>();
        Map<String, String> responseMap = new HashMap<>();
        Map<String, String> requestMap = new HashMap<>();
        try {
            WXPay wxpay = new WXPay(wxPayAppConfig);
            requestMap.put("body", body);                                     // 商品描述
            requestMap.put("out_trade_no", orderNo);                          // 商户订单号
            requestMap.put("total_fee", String.valueOf((int)(amount*100)));   // 总金额
            requestMap.put("spbill_create_ip", HttpContextUtils.getIpAddr()); // 终端IP
            requestMap.put("trade_type", "APP");                              // App支付类型
            requestMap.put("notify_url", wxPayAppConfig.getPayNotifyUrl());   // 接收微信支付异步通知回调地址
            Map<String, String> resultMap = wxpay.unifiedOrder(requestMap);
            //获取返回码
            String returnCode = resultMap.get("return_code");
            String returnMsg = resultMap.get("return_msg");
            //若返回码为SUCCESS，则会返回一个result_code,再对该result_code进行判断
            if ("SUCCESS".equals(returnCode)) {
                String resultCode = resultMap.get("result_code");
                String errCodeDes = resultMap.get("err_code_des");
                if ("SUCCESS".equals(resultCode)) {
                    responseMap = resultMap;
                }
            }
            if (responseMap == null || responseMap.isEmpty()) {
                return ResultMap.error("获取预支付交易会话标识失败");
            }
            // 3、签名生成算法
            Long time = System.currentTimeMillis() / 1000;
            String timestamp = time.toString();
            returnMap.put("appid", wxPayAppConfig.getAppID());
            returnMap.put("partnerid", wxPayAppConfig.getMchID());
            returnMap.put("prepayid", responseMap.get("prepay_id"));
            returnMap.put("noncestr", responseMap.get("nonce_str"));
            returnMap.put("timestamp", timestamp);
            returnMap.put("package", "Sign=WXPay");
            returnMap.put("sign", WXPayUtil.generateSignature(returnMap, wxPayAppConfig.getKey()));//微信支付签名
            return ResultMap.ok().put("data", returnMap);
        } catch (Exception e) {
            logger.error("订单号：{}，错误信息：{}", orderNo, e.getMessage());
            return ResultMap.error("微信支付统一下单失败");
        }
    }

    @Override
    public String notify(String notifyStr) {
        String xmlBack = "<xml><return_code><![CDATA[FAIL]]></return_code><return_msg><![CDATA[报文为空]]></return_msg></xml> ";
        try {
            // 转换成map
            Map<String, String> resultMap = WXPayUtil.xmlToMap(notifyStr);
            WXPay wxpayApp = new WXPay(wxPayAppConfig);
            if (wxpayApp.isPayResultNotifySignatureValid(resultMap)) {
                String returnCode = resultMap.get("return_code");  //状态
                String outTradeNo = resultMap.get("out_trade_no");//商户订单号
                String transactionId = resultMap.get("transaction_id");
                if (returnCode.equals("SUCCESS")) {
                    if (StringUtils.isNotBlank(outTradeNo)) {
                        /**
                         * 注意！！！
                         * 请根据业务流程，修改数据库订单支付状态，和其他数据的相应状态
                         *
                         */
                        logger.info("微信手机支付回调成功,订单号:{}", outTradeNo);
                        xmlBack = "<xml><return_code><![CDATA[SUCCESS]]></return_code><return_msg><![CDATA[OK]]></return_msg></xml>";
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return xmlBack;
    }
    
     @Override
    public ResultMap refund(String orderNo, double amount, String refundReason){

        if(StringUtils.isBlank(orderNo)){
            return ResultMap.error("订单编号不能为空");
        }
        if(amount <= 0){
            return ResultMap.error("退款金额必须大于0");
        }

        Map<String, String> responseMap = new HashMap<>();
        Map<String, String> requestMap = new HashMap<>();
        WXPay wxpay = new WXPay(wxPayAppConfig);
        requestMap.put("out_trade_no", orderNo);
        requestMap.put("out_refund_no", UUIDGenerator.getOrderNo());
        requestMap.put("total_fee", "订单支付时的总金额，需要从数据库查");
        requestMap.put("refund_fee", String.valueOf((int)(amount*100)));//所需退款金额
        requestMap.put("refund_desc", refundReason);
        try {
            responseMap = wxpay.refund(requestMap);
        } catch (Exception e) {
            e.printStackTrace();
        }
        String return_code = responseMap.get("return_code");   //返回状态码
        String return_msg = responseMap.get("return_msg");     //返回信息
        if ("SUCCESS".equals(return_code)) {
            String result_code = responseMap.get("result_code");       //业务结果
            String err_code_des = responseMap.get("err_code_des");     //错误代码描述
            if ("SUCCESS".equals(result_code)) {
                //表示退款申请接受成功，结果通过退款查询接口查询
                //修改用户订单状态为退款申请中或已退款。退款异步通知根据需求，可选
                //
                return ResultMap.ok("退款申请成功");
            } else {
                logger.info("订单号:{}错误信息:{}", orderNo, err_code_des);
                return ResultMap.error(err_code_des);
            }
        } else {
            logger.info("订单号:{}错误信息:{}", orderNo, return_msg);
            return ResultMap.error(return_msg);
        }
    }

}

```

#### 7、定义通用返回结果集 ResultMap

```java
package com.annaru.common.result;

import org.apache.http.HttpStatus;

import java.util.HashMap;
import java.util.Map;

/**
 * @Description 通用返回结果集
 * @Author 
 * @Date 2018/6/12 15:13
 */
public class ResultMap extends HashMap<String, Object> {
    public ResultMap() {
        put("state", true);
        put("code", 0);
        put("msg", "success");
    }

    public static ResultMap error(int code, String msg) {
        ResultMap r = new ResultMap();
        r.put("state", false);
        r.put("code", code);
        r.put("msg", msg);
        return r;
    }

    public static ResultMap error(String msg) {
        return error(HttpStatus.SC_INTERNAL_SERVER_ERROR, msg);
    }

    public static ResultMap error() {
        return error(HttpStatus.SC_INTERNAL_SERVER_ERROR, "未知异常，请联系管理员");
    }

    public static ResultMap ok(String msg) {
        ResultMap r = new ResultMap();
        r.put("msg", msg);
        return r;
    }

    public static ResultMap ok(Map<String, Object> par) {
        ResultMap r = new ResultMap();
        r.putAll(par);
        return r;
    }

    public static ResultMap ok() {
        return new ResultMap();
    }

    public ResultMap put(String key, Object value) {
        super.put(key, value);
        return this;
    }

}
```


title: Java中间件入门教程
date: 2017-12-03 00:00:00
---
<Excerpt in index | 首页摘要> 
中间件：中间件是一种介于操作系统和应用软件之间的一种软件，它使用系统软件所提供的基础服务（功能），衔接网络上应用系统的各个部分或不同的应用，能够达到资源共享、功能共享的目的。
若是以新一代的中间件系列产品来组合应用，同时配合以可复用的商务对象构件，则应用开发费用可节省至80%。 
<!-- more -->
<The rest of contents | 余下全文>
[TOC]


##前言##
本博客介绍Java中间件的一些知识，仅仅是一些知识储备。
##中间件##
###中间件概念###
中间件：中间件是一种介于操作系统和应用软件之间的一种软件，它使用系统软件所提供的基础服务（功能），衔接网络上应用系统的各个部分或不同的应用，能够达到资源共享、功能共享的目的。
若是以新一代的中间件系列产品来组合应用，同时配合以可复用的商务对象构件，则应用开发费用可节省至80%。 
###中间件分类###
 1. 消息中间件
消息中间件适用与进行网络通讯的系统，建立网络通讯的通道，进行数据和文件的传送
产品：ActiveMQ、ZeroMQ、RabbitMQ、IBM webSphere MQ...
 2. 交易中间件
 交易中间件管理分布与不同操作系统的数据，实现数据一致性，保证系统的负载均衡
 产品：IBM CICS,Bea tuxedo...
 3. 对象中间件
 保证不同厂家的软件之间的交互访问
 产品：IBM componentbroker, iona orbix,borland visibroker...
 4. 应用服务器
 用来构造internet/intranet应用和其它分布式构件应用
 产品：IBM Websphere,Bea weblogic...
 5. 安全中间件
 以公钥基础设施（pki）为核心的、建立在一系列相关国际安全标准之上的一个开放式应用开发平台
 产品：entrust entrust...
 6. 应用集成服务器
 把工作流和应用开发技术如消息及分布式构件结合在一起，使处理能方便自动地和构件、script 应用、工作流行为结合在一起，同时集成文档和电子邮件
产品：lss flowman、ibm flowmark、vitria businessagiliti

##ESB##
ESB，即企业服务总线
松散耦合一直是企业软件开发中的一个很重要的内容，而面向服务的SOA编程在随着ESB的应用得到了进一步的发展，ESB就像服务提供者和服务使用者之间的中间层
![这里写图片描述](http://img.blog.csdn.net/20170424134902455?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
##JMS##
JMS，即Java Message Service
ESB仅仅是作为一个中间层，所以应用程序之间的消息通讯必须借助JMS，即通过JMS从服务使用者接收消息，并将其转发到相应的服务提供者。
而且，JMS 还定义了可发送的若干不同类型的消息。例如，Text 消息包含消息的字符串表示形式；Object 消息包含序列化的 Java 对象；Map 消息包含键/值对的映射，等等。

附录：
MQ DEMO：

```
package com.wms.batchMsg;

import java.io.File;
import java.io.IOException;
import java.sql.Timestamp;
import java.text.ParseException;
import java.util.Date;

import org.apache.log4j.Logger;

import com.ibm.mq.MQEnvironment;
import com.ibm.mq.MQException;
import com.ibm.mq.MQGetMessageOptions;
import com.ibm.mq.MQMessage;
import com.ibm.mq.MQPutMessageOptions;
import com.ibm.mq.MQQueue;
import com.ibm.mq.MQQueueManager;
import com.ibm.mq.constants.MQConstants;

public class MQUtil {
	
	private static String qmName;      
    private static MQQueueManager qMgr;
    
    private static Logger logger = Logger.getLogger(MQUtil.class);
    
    static{
    	try{
	        MQEnvironment.hostname=ConfigManager.getValue("MQ_MQHost");
	        MQEnvironment.channel=ConfigManager.getValue("MQ_Server_Channel");
	        MQEnvironment.CCSID=Integer.parseInt(ConfigManager.getValue("MQ_CCSID"));
	        MQEnvironment.port=Integer.parseInt(ConfigManager.getValue("MQ_port"));
	        //MQEnvironment.userID = ConfigManager.getValue("MQ_UserId");
	        //MQEnvironment.password = ConfigManager.getValue("MQ_pass");
	        qmName = ConfigManager.getValue("MQ_QMname");
	        MQEnvironment.properties.put(MQConstants.TRANSPORT_PROPERTY,MQConstants.TRANSPORT_MQSERIES_CLIENT);
	        qMgr = new MQQueueManager(qmName);
		}catch(MQException e){
			e.printStackTrace();
			logger.info("qManager failed: Completion code " + e.completionCode + " Reason Code is "
					+ e.reasonCode);
		}
    }
    
    public static MQQueue getSendQueue(String queueName) {
		MQQueue sQueue;
        int openSendOptions = MQConstants.MQOO_OUTPUT | MQConstants.MQOO_FAIL_IF_QUIESCING
                | MQConstants.MQOO_SET_IDENTITY_CONTEXT;
        try {
            sQueue = qMgr.accessQueue(queueName, openSendOptions);
        } catch (MQException e) {
            e.printStackTrace();
            return null;
        }
        return sQueue;
    }
    
    public static MQQueue getReceiveQueue(String revQueueName){
		MQQueue rQueue ;
		int openRcvOptions = MQConstants.MQOO_INPUT_AS_Q_DEF | MQConstants.MQOO_FAIL_IF_QUIESCING;
		try{
			rQueue = qMgr.accessQueue(revQueueName, openRcvOptions);
		}catch(MQException e){
			e.printStackTrace();
			return null;
		}
		return rQueue;
	}
    
    public static void sendMsg(MQMsgEntity entity,String queueName) {
		MQQueue sendQ = null;
        try {
            MQMessage qMsg = new MQMessage();
            byte[] qByte = entity.getMsgStr().getBytes("UTF-8");
//            String message = entity.getMsgStr();
            qMsg.messageId = MQConstants.MQMI_NONE;
            //TODO send and receive
            if(entity.getCorrelId()!=null){
            	qMsg.correlationId = entity.getCorrelId();
            }
            qMsg.format = MQConstants.MQFMT_STRING;
            qMsg.write(qByte);
            MQPutMessageOptions pmo = new MQPutMessageOptions();
            pmo.options = pmo.options + MQConstants.MQPMO_NEW_MSG_ID;
            pmo.options = pmo.options + MQConstants.MQPMO_NO_SYNCPOINT;
            pmo.options = pmo.options + MQConstants.MQPMO_SET_IDENTITY_CONTEXT;
            sendQ = getSendQueue(queueName);
            sendQ.put(qMsg, pmo);
            qMgr.commit();
            //logger.info("The send message is: " +new String(qByte,"UTF-8"));
        } catch (MQException e) {
            logger.info("A WebSphere MQ error occurred : Completion code "
                            + e.completionCode + " Reason Code is "
                            + e.reasonCode);
        } catch (java.io.IOException e) {
        	logger.info("An error occurred whilst to the message buffer "
                            + e);
        }finally{
        	try{
        		if(sendQ!=null){
        			sendQ.close();
        		}
			}catch(MQException e){
				// TODO Auto-generated catch block
				e.printStackTrace();
				logger.info("Error for MQ connection:"+e.getMessage());
			}
        }

    }
    
//    public static void messageHandlerByQueueName(MQMsgEntity entity,String queueName) {
//    	try {
//    		if(queueName.equalsIgnoreCase("sap_OrdersQueue")){
//        		ECOrder order = new ECOrder();
//        		order.CallOrderCURFC(entity, "ZECI001");
//        	}else if(queueName.equalsIgnoreCase("sap_OrderPendReqQueue")){
//        		ECOrderPending orderPending = new ECOrderPending();
//        		orderPending.CallOrderPendRFC(entity, "ZECI005");
//        	}else if(queueName.equalsIgnoreCase("sap_OrderPendCancelQueue")){
//        		ECOrderPending orderPending = new ECOrderPending();
//        		orderPending.CallCancelOrderPendRFC(entity, "ZECI006");
//        	}else if(queueName.equalsIgnoreCase("sap_ECReturnsQueue")){
//        		ECOrder order = new ECOrder();
//        		order.callOrderCancelRFC(entity, "ZECI001");
//        	}else if(queueName.equalsIgnoreCase("sap_downpaymentQueue")){
//        		ECDownPayment downPayment = new ECDownPayment();
//        		downPayment.callDownPaymentRFC(entity, "ZECI007");
//        	}else if(queueName.equalsIgnoreCase("sap_360LBPQueue")){
//        		EC360LBP lbp = new EC360LBP();
//        		lbp.generateHtmlFromQueue(entity.getMsgStr());
//        	}
//		} catch (Exception e) {
//			e.printStackTrace();
//			logger.error(e.getMessage());
//		}
//    	
//    }
    
    public MQQueueManager generateNewMQQM(){
		MQQueueManager qMgr = null;
		try{
			  
	        MQEnvironment.hostname=ConfigManager.getValue("MQ_MQHost");
	        MQEnvironment.channel=ConfigManager.getValue("MQ_Server_Channel");
	        MQEnvironment.CCSID=Integer.parseInt(ConfigManager.getValue("MQ_CCSID"));
	        MQEnvironment.port=Integer.parseInt(ConfigManager.getValue("MQ_port"));   
	        String qmName = ConfigManager.getValue("MQ_QMname");
	        MQEnvironment.properties.put(MQConstants.TRANSPORT_PROPERTY,MQConstants.TRANSPORT_MQSERIES_CLIENT);
	        qMgr = new MQQueueManager(qmName);
	        
		}catch(MQException e){
			e.printStackTrace();
			logger.info("qManager failed: Completion code " + e.completionCode + " Reason Code is "
					+ e.reasonCode);
		}
		return qMgr;
	}
    
    public void MultiThreadGetMqMessage(MQQueueManager qMgr,String queueName){
    	MQQueue revQ = null;
        String mqString = null;
        MQMsgEntity entity = new MQMsgEntity();
        
        int openRcvOptions = MQConstants.MQOO_INPUT_AS_Q_DEF | MQConstants.MQOO_FAIL_IF_QUIESCING;
        try {
        	MQMessage retrievedMessage = new MQMessage();
            MQGetMessageOptions gmo = new MQGetMessageOptions();
            gmo.options += MQConstants.MQPMO_NO_SYNCPOINT;// 
            gmo.options = gmo.options + MQConstants.MQGMO_WAIT;//
            gmo.options = gmo.options + MQConstants.MQGMO_FAIL_IF_QUIESCING;// 
            gmo.waitInterval = MQConstants.MQWI_UNLIMITED;// 
            gmo.matchOptions = MQConstants.MQMO_MATCH_MSG_ID;
            retrievedMessage.format=MQConstants.MQFMT_STRING;
            //  MQC.MQWI_UNLIMITED;
            revQ = qMgr.accessQueue(queueName, openRcvOptions);
            revQ.get(retrievedMessage, gmo);
            qMgr.commit();
            int length = retrievedMessage.getDataLength();
            if(length >0){
            	long startTime = System.currentTimeMillis();
	            byte[] msg = new byte[length];
	            retrievedMessage.readFully(msg);
	            mqString = new String(msg, "UTF-8");
	            if(queueName.equalsIgnoreCase("sap_360LBPQueue")){
	            	mqString = mqString.replace("'", "\"");
	            }
	            long timeuse = System.currentTimeMillis() - startTime;
	            Date currentDate = new Date();
    			Timestamp receiveTimestamp = new Timestamp(currentDate.getTime());
	            logger.info("=========mqString from "+queueName+" :"+mqString);
	            DBUtil.insertIntoMQLog("Receive",queueName, mqString, timeuse, "success", "", null, receiveTimestamp);
	            entity.setMsgStr(mqString);
	            //messageHandlerByQueueName(entity,queueName);
	            
    			
            }else{
            	logger.info("Error MQ string Sent!");
            }
        }
        catch (MQException e) {
        	e.printStackTrace();
            if (e.reasonCode != 2033) 
            {
            	logger.info(e.getMessage());
                logger.info("Completion code "
                        + e.completionCode + " Reason Code is " + e.reasonCode);
            }
        } catch (IOException e) {
            logger.info("IO error:" + e.getMessage());
        } finally{
        	try{
        		if(revQ!=null){
        			revQ.close();
        		}
			}catch(MQException mqEx){
				int rc = mqEx.reasonCode;
				if (rc != MQException.MQRC_NO_MSG_AVAILABLE)
				{
					logger.info(" PUT Message failed with rc = " 
				+ rc);
				}

			}
        }
    }
    
    public static String getMQMessage(String queueName) throws ParseException {
        MQQueue revQ = null;
        String mqString = null;
        MQMsgEntity entity = new MQMsgEntity();
        try {
        	MQMessage retrievedMessage = new MQMessage();
            MQGetMessageOptions gmo = new MQGetMessageOptions();
            gmo.options += MQConstants.MQPMO_NO_SYNCPOINT;// 
            gmo.options = gmo.options + MQConstants.MQGMO_WAIT;//
            gmo.options = gmo.options + MQConstants.MQGMO_FAIL_IF_QUIESCING;// 
            gmo.waitInterval = MQConstants.MQWI_UNLIMITED;// 
            gmo.matchOptions = MQConstants.MQMO_MATCH_MSG_ID;
            retrievedMessage.format=MQConstants.MQFMT_STRING;
            //  MQC.MQWI_UNLIMITED;
            revQ = getReceiveQueue(queueName);
            revQ.get(retrievedMessage, gmo);
            qMgr.commit();
            int length = retrievedMessage.getDataLength();
            if(length >0){
	            byte[] msg = new byte[length];
	            retrievedMessage.readFully(msg);
	            mqString = new String(msg, "UTF-8");
	            logger.info("=========getMQMessage===mqString from "+queueName+" :"+mqString);
	            entity.setMsgStr(mqString);
	            //messageHandlerByQueueName(entity,queueName);
            }else{
            	logger.info("Error MQ string Sent!");
            }
        }
        catch (MQException e) {
        	e.printStackTrace();
            if (e.reasonCode != 2033) 
            {
                e.printStackTrace();
                logger.info("Completion code "
                        + e.completionCode + " Reason Code is " + e.reasonCode);
            }
        } catch (java.io.IOException e) {
            System.out.println("error" + e.getMessage());
        }finally{
        	try{
        		if(revQ!=null){
        			revQ.close();
        		}
			}catch(MQException mqEx){
				int rc = mqEx.reasonCode;
				if (rc != MQException.MQRC_NO_MSG_AVAILABLE)
				{
				System.out.println(" PUT Message failed with rc = " 
				+ rc);
				}

			}
        }
        return mqString;
    }

	public void revAndSend(MQMsgEntity entity,String queueName){
		//
		sendMsg(entity,queueName);
	}
	
	public void subscribeMessage() throws ParseException{
		while(true){
			logger.info("waiting to get message.....");
			getMQMessage("sap_OrdersQueue");
			
		}
	}
	
	public void subscribeOrderPendMessage() throws ParseException{
		while(true){
			logger.info("waiting to get message.....");
			getMQMessage("sap_ECReturnsQueue");
			
		}
	}
	
	
	
	public  static void main(String[] args) throws IOException, ParseException {
		MQMsgEntity entity = new MQMsgEntity();
		String sendMsg = XMLBeanUtil.readFileToString(new File("D://batchXML0108.txt"));
		int intPktCtlNbr = 1;
		String StrPkt = null;
		String newPktCtlNbr =null;
		for (int i = 0; i < 20000; i++) {
			newPktCtlNbr = String.format("%09d", intPktCtlNbr+i);  
			StrPkt="<PktCtlNbr>"+"V"+newPktCtlNbr+"</PktCtlNbr>";
			String changeSendMsg = sendMsg.replaceAll("<PktCtlNbr>6001996171</PktCtlNbr>", StrPkt);
			System.out.println(StrPkt);
			entity.setMsgStr(changeSendMsg);
			sendMsg(entity,"wms_SAPOrderQueue");
		}
//		MQUtil util = new MQUtil();
//		util.subscribeMessage();
//		util.subscribeOrderPendMessage();
//		util.messageHandlerByQueueName(entity, "sap_360LBPQueue");
//		getMQMessage("sap_OrderPendCancelQueue");
//		System.out.println("rev message is:"+message);
	}

}


```


1. 필요 파일
	* JDK 설치
		- jdk-7u75-linux-i586.gz
	* Apache 설치
		- gcc
		- apr-1.5.1.tar.gz
		- apr-util-1.5.3.tar.gz
		- pcre-8.35.tar.bz2
		- httpd-2.2.29.tar.gz
	* JBOSS 설치		
		- jboss-as-7.1.1.Final.tar.gz
	* postgresql 설치
		- libxslt-1.1.17-4.el5_8.3.i386.rpm
		- postgresql93-libs-9.3.6-1PGDG.rhel5.i386.rpm
		- postgresql93-9.3.6-1PGDG.rhel5.i386.rpm
		- postgresql93-contrib-9.3.6-1PGDG.rhel5.i386.rpm		
		- postgresql93-server-9.3.6-1PGDG.rhel5.i386.rpm
		
2. JDK 설치
	//#yum install java-1.7.0-openjdk-devel.x86_64 // yum 설치시
	#tar -zxvf jdk-7u75-linux-i586.gz   
	#mv  jdk-7u75-linux-i586.gz /usr/local
	#cd /usr/local
	#ln -s  jdk-7u75-linux-i586.gz java
	#vi /etc/profile
		JAVA_HOME=/usr/local/java
		CLASSPATH=.:$JAVA_HOME/lib/tools.jar
		PATH=$PATH:$JAVA_HOME/bin
		export JAVA_HOME CLASSPATH PATH
	#mv /usr/bin/java /usr/bin/java-old
	#source /etc/profile
	
3. Apache 설치
	* apr 설치
	#tar xvfz apr-1.5.1.tar.gz
	#cd apr-1.5.1
	#./configure --prefix=/usr/local/apr
	#make
	#make install
	* apr-util 설치
	#tar xvfz apr-util-1.5.3.tar.gz
	#cd apr-util-1.5.3
	#./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
	#make
	#make install
	* pcre 설치
	#tar xvf pcre-8.35.tar.bz2
	#cd pcre-8.35
	#./configure --prefix=/usr/local
	#make
	#make install
	* Apache 설치
	#tar xvfz httpd-2.2.29.tar.gz
	#./configure  --prefix=/usr/local/apache \
					--enable-rule=SHARED_CORE  \
					--enable-so  \
					--enable-rewrite  \
					--enable-vhost-alias  \
					--enable-ssl  \
					--enable-proxy  \
					--enable-shared=max  \
					--enable-modules=shared  \
					--enable-mods-shared=all  \
					--with-apr=/usr/local/apr  \
					--with-charset=utf-8  \
					--with-apr-util=/usr/local/apr-util
	#make
	#make install
	#vi /usr/local/apache/conf/httpd.conf
		server_name 수정 (없으면 IP를 적어라)
	#/usr/local/apache/bin/apachectl restart
	#cp /usr/local/apache/bin/apachectl /etc/init.d/httpd
	#chmod 755 /etc/init.d/httpd

4. JBOSS 설치
	#tar xvfz jboss-as-7.1.1.Final.tar.gz
	#mv jboss-as-7.1.1.Final /usr/local/jboss	
	#cp /usr/local/jboss/bin/init.d/jboss-as-standalone.sh /etc/init.d/jboss
	#mkdir /etc/jboss-as
	#cp /usr/local/jboss/bin/init.d/jboss-as.conf /etc/jboss-as/
	#vi /etc/jboss-as/jboss-as.conf  (환경 세팅하는 곳)
		#JBOSS_USER=jboss-as 을 JBOSS_USER=jboss(계정명) 으로 수정
		JBOSS_HOME=/usr/local/jboss <-- 로 path 수정
		
	#/usr/local/jboss/bin/add-user.sh ktsfs 1234qwer  (계정등록)	
	#vim /usr/local/jboss/standalone/configuration/standalone.xml 
	(http 포트수정 9109, 관리포트수정 9209)
	<alias name="localhost"> 다음라인에 <alias name="10.217.37.155"> 추가 ( 자신이 이것들을 허용하겠다라는 것)
	
5. PostgreSQL 설치 (root 계정 필요, root계정 없을시 소스에서 빌드해야 함)
	#rpm -Uvh libxslt-1.1.17-4.el5_8.3.i386.rpm
	#rpm -Uvh postgresql93-libs-9.3.6-1PGDG.rhel5.i386.rpm
	#rpm -Uvh postgresql93-9.3.6-1PGDG.rhel5.i386.rpm
	#rpm -Uvh postgresql93-contrib-9.3.6-1PGDG.rhel5.i386.rpm		
	#rpm -Uvh postgresql93-server-9.3.6-1PGDG.rhel5.i386.rpm
	#/etc/init.d/postgresql9.3 initdb

== 리눅스 서비스 등록 하기
	#chkconfig --add /etc/init.d/서비스명
	#service 서비스명 start/stop/restart/status ...
	
== JBOSS 설정및 스타트 정상여부 확인	
	#yum install telnet 
	#telnet localhost 8109
	GET / HTTP/1.0 (엔터두번)
	200OK 떨어져야함  ( 400 클라문제 500대면 서버문제)
	
------------------------------------------------------------------------------------------------------------------------
MCSAuth.java
	public String makeCryptograph_EUC_KR(String data, String authKeyValue){
		String token = "";
		
		try {			
			token = encrypto2(data.getBytes("EUC-KR"), authKeyValue.getBytes());	
		} catch (Exception e) {
			e.printStackTrace();
		}		
		
		return token;
	}
-------------------------------------------
XmlMaking.java
package com.kt.xroshot.xml;

import java.util.ArrayList;

import com.kt.xroshot.auth.AuthKey;
import com.kt.xroshot.auth.MCSAuth;

public class XmlMaking {
	
	AuthKey authKey = null;
	MCSAuth mcsAuth = null;
	String authKeyValue = "";
	int authKeyIndex = -1;
	String oneTimeKeyValue = "";
	
	public XmlMaking(String cert) {
		this.authKey = new AuthKey();
		this.authKey.getAuthKey(cert);		
		this.mcsAuth = new MCSAuth();
		this.authKeyValue = this.authKey.getAuthKey();
		this.authKeyIndex = this.authKey.getIndex();		
	}	

	public String getServerTime(String id){		 
		String serverTimeRequest = "<?xml version='1.0' encoding='UTF-8'?>\r"
								 + "  <MAS method='req_auth'>\r"
								 + "  <ServiceProviderID>" + id + "</ServiceProviderID>\r"
								 + "</MAS>\0";				
				
		return serverTimeRequest.replace("'", "\"");
	}
	
	public String getRequestLoginXml(String id, String pw, String endUserID, String serverTime, String oneTimeKey){
				
		String authTicket = this.authKey.makeAuthTicket(id, pw, endUserID, serverTime, oneTimeKey);		
		String eEncodedAuthTicket = mcsAuth.makeCryptograph(authTicket, this.authKeyValue);	
		this.oneTimeKeyValue = oneTimeKey;
		String login = "<?xml version='1.0' encoding='UTF-8'?>\r"
					 + "<MAS method='req_regist'>\r"
					 + "  <ServiceProviderID>" + id + "</ServiceProviderID>\r"
					 + "  <EndUserID>" + endUserID + "</EndUserID>\r"
					 + "  <AuthTicket>"  + eEncodedAuthTicket + "</AuthTicket>\r"								
					 + "  <AuthKey>" + Integer.toString(authKeyIndex) + "</AuthKey>\r"
					 + "  <Version>1.0.15</Version>\r"
					 + "</MAS>\0";
		
		return login.replace("'", "\"");					
	} 
	
	public String getSms(String callbackNumber, String customMessageID,
			ArrayList<String> receiveNumbers, String subject, String content){
		
		String eCallbackNumber = mcsAuth.makeCryptograph(callbackNumber, this.oneTimeKeyValue);
		String eContent = mcsAuth.makeCryptograph_EUC_KR(content, this.oneTimeKeyValue);		
		String receiveNumber = "";
		
		for (int i=0; i<receiveNumbers.size(); i++){
			String phoneNumber = receiveNumbers.get(i);
			String ePhoneNumber = mcsAuth.makeCryptograph(phoneNumber, this.oneTimeKeyValue);
			receiveNumber += "    <ReceiveNumber seqNo='" + (i+1) + "'>" + ePhoneNumber + "</ReceiveNumber>\r";		
		}
		 
		String sms =  "<?xml version='1.0' encoding='UTF-8'?>\r"
					+ "<MAS method='req_send_message_2'>\r"
					+ "  <MessageType>1</MessageType>\r"
					+ "  <MessageSubType>1</MessageSubType>\r"
					+ "  <CallbackNumber>" + eCallbackNumber +  "</CallbackNumber>\r"
					+ "  <CustomMessageID>" + customMessageID + "</CustomMessageID>\r"
					+ "  <Message>\r"
					+ 		receiveNumber 
					+ "    <Subject>" + subject + "</Subject>\r"
					+ "    <Content>" + eContent + "</Content>\r"
					+ "  </Message>\r"
					+ "</MAS>\0";
		
		return sms.replace("'", "\"");
	}
	
	public String getAlive(){
		String alive = "<?xml version='1.0' encoding='UTF-8'?>\r"
					 + "<MAS method='req_ping'>\r"
					 + "  <Result>0</Result>\r"
					 + "</MAS>\0";
		
		return alive.replace("'", "\"");
	}
	
	public String getLogOut(){
		String logOut = "<?xml version='1.0' encoding='UTF-8'?>\r"
				 + "<MAS method='req_unregist'>\r"
				 + "  <Result>0</Result>\r"
				 + "</MAS>\0";
	
		return logOut.replace("'", "\"");
	} 
}




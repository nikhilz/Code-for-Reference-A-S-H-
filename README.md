# Code-for-Reference-A-S-H-

Fetching details from Database

(use these annotions above that class)
@Repository
@EnableTransactionManagement 
(also create instance of sessionFactory in that class)

----------------------Get user details by ID---------------------------
--using Criteria

	

	@Transactional
	public AdminUserLogin getUserLoginBYUserId(Long Id) {
		Criteria cr = sessionFactory.getCurrentSession().createCriteria(AdminUserLogin.class);
        cr.add(Restrictions.eq("isActive",FrameworkAppConstants.CONSTANT_YES ));
        cr.add(Restrictions.eq("userId",Id));
        List<AdminUserLogin> lst = cr.list();
        if (lst.isEmpty()) {
            return null;
        } else {
            return lst.get(0);
        }

--using HQL

	@Transactional
	@Override
	public Customer getDetails(String UserInfo) {
		System.out.println("----------getDetailsByCustID----------------");
		Session session = sessionFactory.getCurrentSession();
		   Query q = session.createQuery("from Customer where customerIdNum = :customerIdNum ");
		   q.setParameter("customerIdNum", UserInfo);
		   //Customer cust=(Customer) q.list().get(0);
		   List<Customer> lst= q.list();
		   if (lst.isEmpty()) {
	        	System.out.println("DAO result is null");
	            return null;
	        } else {
	        	System.out.println("DAO result is not null "+lst.size());
	            return lst.get(0);
	        }



----------------------- save User Details ------------------------------------

	@Transactional
	public void saveUserGeneralDetails(AdminUserGeneralDetails adminUserGeneralDetails) {
		sessionFactory.getCurrentSession().saveOrUpdate(adminUserGeneralDetails);
		
	}

----------------------- Controller for search User ----------------------------

	@RequestMapping(value = "/searchUser", method = RequestMethod.POST)
	public ModelAndView searchUserInfo(@RequestBody String payload) {

		System.out.println("inside searchUser ------>" + payload);

		ModelAndView mav = new ModelAndView("CustInfo");
		try {
			JSONObject reqJson = new JSONObject(payload);
			if (!reqJson.has("user_info") || reqJson.get("user_info") == null
					|| "".equals(reqJson.get("user_info").toString().trim())) {
				throw new CustomNonFatalException("info of the User not entered", "info of the User not entered");
			}

			String searchUser = reqJson.get("user_info").toString();

		/*	if (Pattern.matches("^(?![0-9]*$)[a-zA-Z0-9]{8,}$", searchUser)) {
				System.out.println("---Its user name---");
				try {
					UserLogin userlogin = customerInformationServiceImpl.getDetailsByUserId(searchUser);

					mav.addObject("customerDetails", userlogin);
					System.out.println("userlogin-->" + userlogin.toString());
				} catch (IndexOutOfBoundsException e) {
					// If entered UserName is not in DB
					System.out.println("User Not found");
					mav.addObject("status", 404);

				}

			} else */
				if (Pattern.matches("^[0-9]{6,8}$", searchUser)) { //RegEx
				System.out.println("---Its CustID---");
				try {
					Customer customer = customerInformationServiceImpl.getDetails(searchUser);

					System.out.println("from customer-->" + customer.getCustPartyId());
					Party party = customerInformationServiceImpl.getDetailsFromParty(customer.getCustPartyId());
					
					CustomerInformationVO customerInformationVO = new CustomerInformationVO();
					customerInformationVO.setAccountIdNum(customer.getAccountIdNum());
					customerInformationVO.setPartyType(party.getPartyType().getPartyType());
					customerInformationVO
							.setPartyClassificationId(party.getPartyClassification().getPartyClassificationId());
					mav.addObject("customerDetails", customerInformationVO);
					mav.addObject("status", 200);
				} catch (Exception e) {
					// If entered CustID is not in DB
					System.out.println("User Not found");
					mav.addObject("status", 404);

				}
			} else {
				System.out.println("invalid username");
				mav.addObject("status", 404);
			}

		} catch (Exception e) {
			mav.addObject("status", 300);
			mav.addObject("message", "Unable to Fetch data. Please try again");
			e.printStackTrace();
		}
		return mav;

	}



--------------------------------Angularjs controller for searching User on Submit ------------------------


	$scope.accountIdNum="";
	$scope.partyType="";
	$scope.partyClassificationId=""
	$scope.isUserFound=true;
	$scope.showDetails=false;
	

	$scope.searchUser=function(){
		//var user_info=$scope.user_info;
		var callURL="searchUser.json";
		var data={
				user_info:$scope.user_info
		}
		$http({
			method : 'POST',
			url: callURL,
			data: data,
			cache: false		
	
		}).then(function successCallback(response) {
			if(response.data.status==200){
			console.log(response.data.customerDetails);
			$scope.isUserFound=true;
			$scope.showDetails=true;
			
			$scope.accountIdNum=response.data.customerDetails.accountIdNum;
			$scope.partyType=response.data.customerDetails.partyType;
			$scope.partyClassificationId=response.data.customerDetails.partyClassificationId;	
			
			
			}else{
				$scope.isUserFound=false;
				$scope.showDetails=false;
			}
			
		}, function errorCallback(response) {
			console.log(response.statusText);
		});
	}


-------------------------------- Custom Exception ----------------------------------------------

package com.cmss.fullerton.components.core.util.excepton;

public class CustomNonFatalException extends Exception{
	 public CustomNonFatalException() {
	       super( "Session Exception" );
	    }

	    public CustomNonFatalException( String message,Integer intMessage ,Exception ex) {
	       super(intMessage.toString());
	    }
	    public CustomNonFatalException( String message,Integer intMessage) {
	       super(intMessage.toString());
	    }
	    public CustomNonFatalException( String strMessage,String strMessageId) {
	       super(strMessageId);
	    }
	     public CustomNonFatalException( String strMessage) {
	       super(strMessage);
	    }
	}


-------------------------------- Date Utility --------------------------------------------------

mport java.sql.Timestamp;
import java.text.SimpleDateFormat;
import java.util.Hashtable;
import java.util.Locale;

import com.cmss.fullerton.components.core.util.excepton.CustomNonFatalException;

import java.text.DateFormat;
import java.text.ParseException;
import java.util.Calendar;
import java.util.Date;
import java.util.GregorianCalendar;

/**
 *
 * @author DELL
 */
public class DateUtility {

    public static final String DAYS = "DAYS";
    public static final String HOURS = "HRS";
    public static final String MINUTES = "MIN";
    public static final String SECONDS = "SEC";
    public final static long SECOND_MILLIS = 1000;
    public final static long MINUTE_MILLIS = SECOND_MILLIS * 60;
    public final static long HOUR_MILLIS = MINUTE_MILLIS * 60;
    public final static long DAY_MILLIS = HOUR_MILLIS * 24;
    public final static long YEAR_MILLIS = DAY_MILLIS * 365;

    public static Hashtable hashTab;

    public DateUtility() {
        super();
    }

    public static Timestamp getCurrentDate() {
        java.util.Date today = new java.util.Date();
        return new java.sql.Timestamp(today.getTime());
    }

    public static Timestamp addDays(Timestamp timestamp, int intDays) {
        Calendar c1 = GregorianCalendar.getInstance();
//        System.out.println(timestamp.toString());
//        System.out.println(timestamp.getYear());
        c1.set(Integer.parseInt(timestamp.toString().substring(0, 4)), timestamp.getMonth(), timestamp.getDate()); // 1999 jan 20
        c1.add(Calendar.DATE, intDays);
        java.util.Date date = c1.getTime();
        return new java.sql.Timestamp(date.getTime());
    }

    public static Timestamp stripTime(Timestamp timestamp) {
        return timestamp.valueOf(timestamp.toString().substring(0, 10) + " 00:00:00.000");
    }

    public static Timestamp appendTime(Timestamp timestamp) {
        return timestamp.valueOf(timestamp.toString().substring(0, 10) + " 23:59:59.000");
    }

    public static Timestamp parseStringToDate(String strDate)
            throws CustomNonFatalException {
        DateFormat df = new SimpleDateFormat("MM/dd/yyyy");
        try {
            java.util.Date today = df.parse(strDate);
            return new java.sql.Timestamp(today.getTime());
        } catch (ParseException e) {
//            e.printStackTrace();
//            CustomLogger.error("Unable to parse Date",e);
            throw new CustomNonFatalException("Unable to parse Date", "Invalid date format");
        }

    }

    public static Timestamp parseStringToDate(String strDate, String strFormat)
            throws CustomNonFatalException {
        DateFormat df = new SimpleDateFormat(strFormat);
        try {
            java.util.Date today = df.parse(strDate);
            return new java.sql.Timestamp(today.getTime());
        } catch (ParseException e) {
            throw new CustomNonFatalException("Unable to parse Date", "Invalid date format");
        }

    }

    public static String parseDateToString(java.util.Date today) {
        DateFormat df = new SimpleDateFormat("dd-MMM-yy");
        return df.format(today);
    }
    
    public static String parseDateToString(java.util.Date today, String format) {
        DateFormat df = new SimpleDateFormat(format);
        return df.format(today);
        
    }
    

    public static Timestamp parseDateToDate(Date strDate)
            throws CustomNonFatalException {

        try {

            return new java.sql.Timestamp(strDate.getTime());
        } catch (Exception e) {
//            e.printStackTrace();
//            CustomLogger.error("Unable to parse Date", e);
            throw new CustomNonFatalException("Unable to parse Date", "Invalid date format");
        }

    }

    public static Integer getTimeInInteger() {
        java.util.Date today = new java.util.Date();
        DateFormat df = new SimpleDateFormat("kkmmssSSS");
        return Integer.parseInt(df.format(today));
    }

    public static String formatDate(Timestamp tDate, int iDateFormat, Locale userLocale) throws CustomNonFatalException {
        DateFormat df = null;

        if (tDate == null) {
            return ("");
        }

        if (iDateFormat < 3) {
            df = DateFormat.getDateInstance(DateFormat.SHORT, userLocale);
        } else {
            df = DateFormat.getDateInstance(DateFormat.MEDIUM, userLocale);
        }

        try {
            String formattedDate = df.format(tDate);
            return formattedDate;
        } catch (IllegalArgumentException e) {
            return ("");
        }

    }

    public static long getDaysDifference(Timestamp time1, Timestamp time2, String strType) {

        long timeDifference = time1.getTime() - time2.getTime();

//    System.out.println(" DATEUTIL:"+strType+"_"+timeDifference);
        if (strType.equals(DAYS)) {
            time1 = stripTime(time1);
            time2 = stripTime(time2);
            timeDifference = time1.getTime() - time2.getTime();
            timeDifference = timeDifference / (24 * 60 * 60 * 1000);
        } else if (strType.equals(HOURS)) {
            timeDifference = timeDifference / (60 * 60 * 1000);
        } else if (strType.equals(MINUTES)) {
            timeDifference = timeDifference / (60 * 1000);
        } else if (strType.equals(SECONDS)) {
            timeDifference = timeDifference / (1000);
        } else {
            timeDifference = 0L;
        }
        //  System.out.println(" DATEUTIL:"+strType+"_"+timeDifference);
        return timeDifference;
    }


    
    public static int hoursDiff(Date earlierDate, Date laterDate) {
        if (earlierDate == null || laterDate == null) {
            return 0;
        }

        return (int) ((laterDate.getTime() / HOUR_MILLIS) - (earlierDate.getTime() / HOUR_MILLIS));
    }

}



----------------------------- Web.xml ---------------------------------------------

<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
	</welcome-file-list>
	<!-- The definition of the Root Spring Container shared by all Servlets 
		and Filters -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/config/spring/applicationContext.xml</param-value>
	</context-param>

	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!-- Spring Security -->
	<!--  
	<filter>
		<filter-name>springSecurityFilterChain</filter-name>
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>springSecurityFilterChain</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>-->
	<!-- Processes application requests -->
	<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/config/dispatcher-servlet.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>*.json</url-pattern>
	</servlet-mapping>
	<mime-mapping>
    <extension>ico</extension>
    <mime-type>image/x-icon</mime-type>
</mime-mapping>
	

</web-app>



------------------------------dispatcher. servlet -----------------------------------

<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
	
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven />

	<context:annotation-config />

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources mapping="/resources/**" location="/resources/" />

	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->

	<beans:bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
	  <beans:property name="order" value="1" />
	  <beans:property name="mediaTypes">
		<beans:map>
		   <beans:entry key="json" value="application/json" />
		   <beans:entry key="xml" value="application/xml" />
		   <beans:entry key="rss" value="application/rss+xml" />
		</beans:map>
	  </beans:property>

			  <beans:property name="defaultViews">
		<beans:list>
		  <!-- JSON View -->
		  <beans:bean
			class="org.springframework.web.servlet.view.json.MappingJackson2JsonView">
		  </beans:bean>

		 </beans:list>
	  </beans:property>
	  <beans:property name="ignoreAcceptHeader" value="true" />


	</beans:bean>


	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="order" value="2" />
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".html" />
	</beans:bean>
	
	 
	 
	<context:component-scan base-package="com.cmss.fullerton" />

	
</beans:beans>



----------------------------------ApplicationContext.xml (SQL Server Config) ----------------------------------

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd">

	<!-- Root Context: defines shared resources visible to all other web components -->

	<bean id="myDataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"></property>
		<property name="url" value="jdbc:sqlserver://192.168.0.10:1433;databaseName=fullertoncsp"></property>
		<property name="username" value="sa"></property>
		<property name="password" value="Cyber@987"></property>
	</bean>
	
	<bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<property name="dataSource" ref="myDataSource"></property>
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</prop>
				<prop key="hibernate.hbm2ddl.auto">update</prop>
			</props>
		</property>
		<property name="packagesToScan">
			<list>
				<value>com.cmss.fullerton.bean</value>
			</list>
		</property>
	</bean>
	  
	<bean id="transactionManager"
		class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

	<!--   <tx:annotation-driven /> -->

</beans>




----------------------------------pom.xml required Dependancies--------------------------------- 


<properties>
		<java-version>1.8</java-version>
		<org.springframework-version>4.1.6.RELEASE</org.springframework-version>
		<spring.security.version>3.2.9.RELEASE</spring.security.version>
		<org.aspectj-version>1.6.10</org.aspectj-version>
		<org.slf4j-version>1.6.6</org.slf4j-version>
	</properties>
	<dependencies>
		<!-- Spring -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${org.springframework-version}</version>
			<exclusions>
				<!-- Exclude Commons Logging in favor of SLF4j -->
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc-portlet</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-oxm</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-messaging</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jms</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-instrument-tomcat</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-instrument</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-expression</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-beans</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aspects</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>

		<!-- Spring Security related jars -->

		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-web</artifactId>
			<version>${spring.security.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-config</artifactId>
			<version>${spring.security.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-core</artifactId>
			<version>${spring.security.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-taglibs</artifactId>
			<version>${spring.security.version}</version>
		</dependency>

		<!-- JSON jar -->
		
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20160810</version>
</dependency>
		
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.5.3</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
		<!-- <dependency> <groupId>mysql</groupId> <artifactId>mysql-connector-java</artifactId> 
			<version>5.1.38</version> </dependency> -->

		<!-- SQLJDBC 4 Driver Run below command if you get "Missing artifact com.microsoft.sqlserver:sqljdbc4:jar:4.0" 
			Error mvn install:install-file -Dfile=sqljdbc4.jar -DgroupId=com.microsoft.sqlserver 
			-DartifactId=sqljdbc4 -Dversion=4.0 -Dpackaging=jar -->

		<!-- https://mvnrepository.com/artifact/com.microsoft.sqlserver/mssql-jdbc -->
		<dependency>
			<groupId>com.microsoft.sqlserver</groupId>
			<artifactId>mssql-jdbc</artifactId>
			<version>6.1.0.jre7</version>
		</dependency>


		<!-- Hibernate -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>4.3.2.Final</version>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>4.3.2.Final</version>
		</dependency>

		<!-- AspectJ -->
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjrt</artifactId>
			<version>${org.aspectj-version}</version>
		</dependency>

		<!-- Logging -->
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${org.slf4j-version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>jcl-over-slf4j</artifactId>
			<version>${org.slf4j-version}</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${org.slf4j-version}</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.15</version>
			<exclusions>
				<exclusion>
					<groupId>javax.mail</groupId>
					<artifactId>mail</artifactId>
				</exclusion>
				<exclusion>
					<groupId>javax.jms</groupId>
					<artifactId>jms</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jdmk</groupId>
					<artifactId>jmxtools</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jmx</groupId>
					<artifactId>jmxri</artifactId>
				</exclusion>
			</exclusions>
			<scope>runtime</scope>
		</dependency>

		<!-- @Inject -->
		<dependency>
			<groupId>javax.inject</groupId>
			<artifactId>javax.inject</artifactId>
			<version>1</version>
		</dependency>

		<!-- Servlet -->
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.5</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>jsp-api</artifactId>
			<version>2.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>

		<!-- Test -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.7</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>antlr</groupId>
			<artifactId>antlr</artifactId>
			<version>2.7.7</version>
		</dependency>
	</dependencies>






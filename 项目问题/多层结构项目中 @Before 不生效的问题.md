## 多层结构项目中 @Before 不生效的问题
今天碰到一个问题，在数据库操作删除数据时，碰到其他表需要用到这个表的数据当外键，因此出现删除失败的问题。这个问题的解决方案就是在调用`CrudRepositoy`的`delete()`方法前，先将其他表中相关数据也删除即可。
如果只是在使用的地方简单加删除代码，这样也可以做，但是需要重复加代码，而且不符合一个程序员优秀的变编程习惯，因此这边采用 AOP 的方法，对方法调用进行拦截，在方法调用前将其他表的数据删除，使用 Spring 的自动管理来实现，而不是我们主动去调用。
首先先写下面一段代码：
```
package edu.ecnu.yjsy.py.common;

import edu.ecnu.yjsy.py.model.cultivation.CultivationPlan;
import edu.ecnu.yjsy.py.model.cultivation.CultivationPlanAudit;
import edu.ecnu.yjsy.py.model.cultivation.CultivationPlanRequest;
import edu.ecnu.yjsy.py.repository.cultivation.CultivationPlanAuditRepository;
import edu.ecnu.yjsy.py.repository.cultivation.CultivationPlanRequestRepository;
import org.aspectj.lang.JoinPoint;
]import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

/**
 * @author LewisYoung
 * @created 2020/6/2
 */
@Aspect
@Component
public class CultivationPlanChangeAspect {

    private static final Logger LOG = LoggerFactory.getLogger(CultivationPlanChangeAspect.class);

    @Autowired
    private CultivationPlanAuditRepository auditRepository;

    @Autowired
    private CultivationPlanRequestRepository requestRepository;

    @Pointcut("execution(public * edu.ecnu.yjsy.py.repository.cultivation.CultivationPlanRepository.delete(*))")
    public void interceptDelete(){

    }

    @Before(value = "interceptDelete()")
    public void preDelete(JoinPoint joinPoint){
        LOG.error("pre");
    }

    @Before(value = "call(public * edu.ecnu.yjsy.py.repository.cultivation.CultivationPlanRepository.delete(*)) && args(cultivationPlan)")
    public void interceptCultivationPlanDelete(JoinPoint point, CultivationPlan cultivationPlan){
        LOG.info("培养计划删除前准备切面启动");
//        preCultivationPlanDelete(cultivationPlan);
    }

    // 在删除培养计划之前对审批、申请等内容进行删除，避免外键问题造成删除失败
    @Transactional
    public void preCultivationPlanDelete(CultivationPlan cultivationPlan){
        List<CultivationPlanRequest> requests = requestRepository.findRequestsByStudent(cultivationPlan.getStudent());
        requests.forEach(request -> {
            List<CultivationPlanAudit> audits = auditRepository.findPlanAuditsByPlanRequest(request);
            auditRepository.deleteAll(audits);
        });
        requestRepository.deleteAll(requests);
    }
}
```
然后运行，发现根本无法对方法起到拦截作用，然而在其他项目中是可以拦截的，于是开始排查问题。
首先，检查配置文件。在 resources 文件下建立`aop-bean-py.xml`，内容如下：
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="cultivationPlanChangeAspect"
          class="edu.ecnu.yjsy.py.common.CultivationPlanChangeAspect"
          factory-method="aspectOf" autowire="byType">
    </bean>
</beans>
```
运行后仍然无效，再看配置类。
```
package edu.ecnu.yjsy.py.conf;

import org.springframework.boot.autoconfigure.domain.EntityScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.context.annotation.ImportResource;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@Configuration
@EntityScan("edu.ecnu.yjsy.py.model")
@EnableJpaRepositories("edu.ecnu.yjsy.py.repository")
@ImportResource("classpath:aop-beans-py.xml")
@EnableAspectJAutoProxy(exposeProxy = true)
public class PyModelConfig {

}
```
发现无法通过这里连接到 xml 文件，检查项目的 iml 文件查看项目结构，增加 resources 文件目录。
```
 <sourceFolder url="file://$MODULE_DIR$/src/main/resources" type="java-resource" />
```
添加后仍然无效。。。到这里开始换怀疑 Spring 是否成功创建容器和管理了，于是增加构造函数，检查容器创建是否成功。
```
	public CultivationPlanChangeAspect(){
        LOG.info("CultivationPlanChangeAspect 构造方法启动...");
    }
```
发现项目启动时构造函数输出成功。。。醉了。。。接下来怀疑`@Before`的`value`值是否写对。（这边是最不想怀疑的，因为我之前拿相同的代码在其他切面中是有用的）
将拦截的函数改成 service 层的也没用。。。


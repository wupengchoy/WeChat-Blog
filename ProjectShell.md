```tex
#!/bin/bash
#export JAVA_HOME=/app/jdk/jdk1.8.0_25
#export PATH=$PATH:$JAVA_HOME/bin
 
#java虚拟机启动参数  
JAVA_OPTS="-ms1024m -mx1024m -Xmn256m -XX:MaxPermSize=128m"
 
#需要启动的Java主程序（main方法类）  
APP_MAINCLASS=XXX-server.jar
 
#进行jar包路径的添加
if test -n "$2" 
        then    $APP_MAINCLASS=$2
fi
 
echo "The path of jar is $APP_MAINCLASS"
 
#初始化psid变量（全局）  
psid=0
 
checkpid() {  
   javaps=`jps -l | grep $APP_MAINCLASS`  
   
   if [ -n "$javaps" ]; then  
      psid=`echo $javaps | awk '{print $1}'`  
   else
      psid=0  
   fi  
}  
 
start() {  
   checkpid  
   
   if [ $psid -ne 0 ]; then  
      echo "================================"  
      echo "info: $APP_MAINCLASS already started! (pid=$psid)"  
      echo "================================"  
   else  
      echo -n "Starting $APP_MAINCLASS ..."  
      `nohup java $JAVA_OPTS -jar $APP_MAINCLASS >/dev/null 2>&1 &`
      checkpid  
      if [ $psid -ne 0 ]; then  
         echo "(pid=$psid) [OK]"  
      else  
         echo "[Failed]"  
      fi  
   fi  
}  
 
stop() {  
   checkpid  
   
   if [ $psid -ne 0 ]; then  
      echo -n "Stopping $APP_MAINCLASS ...(pid=$psid) "
      `kill -9 $psid`  
      if [ $? -eq 0 ]; then  
         echo "[OK]"  
      else  
         echo "[Failed]"  
      fi  
   
      checkpid  
      if [ $psid -ne 0 ]; then  
         stop  
      fi  
   else  
      echo "================================"  
      echo "info: $APP_MAINCLASS is not running"  
      echo "================================"  
   fi  
}
 
 
case "$1" in  
   'start')  
      start  
      ;;  
   'stop')  
     stop  
     ;;  
   'restart')  
     stop  
     start  
     ;;  
  *)  
echo "Usage: $0 {start|stop|restart}"  
exit 1  
esac   
exit 0    


```


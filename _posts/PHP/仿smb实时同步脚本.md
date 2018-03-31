---
title: 仿smb实时同步脚本
date: 2017-01-08
tags: [php]
category: [PHP]
---
由于目前公司使用的docker机器，samba特别慢，对于我们这种非vim党、被windows宠坏的是特别痛苦的。
以前一直是强行忍受着samba的延迟，每次保存都需要经过漫长的等待才能同步到docker上，这对于我们这种急性子程序猿简直要了老命。**为了解决这个问题，我开发了一个脚本，分为发送端send.php和接收端recv.php。**发送端部署在windows上，实时查询文件、文件夹的增删改操作。把操作的结果发送给接收端，docker接收端接收到后实时同步结果。**从此解放双手，告别延迟**
<!--more-->
#### 使用方式
- windows上安装php环境、svn。一定要配置好php环境变量。
- recv.php为接收端，需要放置在docker机器上并且能够访问。然后浏览器输入：`docker机器域名:8086/recv.php` 看是不是会出现`hello xhyj!!!`这几个字。没有的话看看路径是不是放错了。
- send.php为发送端，放置在windows上。
- svn co 你需要的代码。比如放在 E:\waimai_svn\antispam。
- 执行`php send.php recv的url windows路径 linux路径`  比如(`php send.php http://gzhxy-xhyj_iwm.docker.iwm.name:8159/recv.php E:\waimai_svn\antispam  /home/map/odp_cater/app/antispam`)
 
#### 原理&总结
- 0、send.php为windows下的发送端，recv.php为linux下的接收端。
- 1、如果查询文件夹是不是删除？重命名？新增？通过记录所有的文件夹名字，每次查询后都与上次进行对比。缺则删、多则删除，重命名则先删原名的文件夹后增新名字的文件夹。
- 2、如何判断文件是不是修改？实时查询文件的修改时间，并与上次的修改时间对比，如果不一样那么就说明文件进行了修改，那么就需要进行windows与linux文件同步。
- 3、任何遍历文件、文件夹？层次遍历。
- 4、使用php内置curl函数簇，发送post请求来实时同步文件的。因为get请求对发送的内容是有长度限制，传统ie为2048个字符。所以当修改文件比较大，可能会超过这个值，因此get不符合我们的需求场景。而post请求没有长度限制。
- 5、usleep是为了防止cpu爆满、unset是为防止内存泄漏。
 
#### 贴代码
 
###### 1、send.php
 
```
<?php
 
set_time_limit(0);
ini_set('memory_limit', '10G');
 
$recv_url =  isset($argv[1])?$argv[1]:'';
$windows_dir = isset($argv[2])?$argv[2]:'';
$linux_dir = isset($argv[3])?$argv[3]:'';
 
SMB::CONF($recv_url,$windows_dir,$linux_dir)->run();
 
class SMB{
   static private $recv_url = "";
    private static $windows_dir = "";
    private static $linux_dir = "";
   private $operate_arr = array('delete_file'=>1,
                           'update_file'=>3,
                           'delete_dir'=>4,
                           'add_dir'=>5);
   private $result = array();
    public function run(){
      $ret_old = $this->read_all_dir(self::$windows_dir);
      unset($this->result);
        while (1){
         $ret_new = $this->read_all_dir(self::$windows_dir);
         unset($this->result);
         //区分文件夹的不同
         $this->diff_dir($ret_old,$ret_new);
         //区分文件的不同
         $this->diff_file($ret_old,$ret_new);
         $ret_old = $ret_new;
         usleep(500000);
        }
    }
   //区分文件的不同
   private function diff_file($ret_old,$ret_new){
      if(empty($ret_old) && empty($ret_new)){
         return false;
      }
      $diff_old = array_diff_assoc($ret_old['file'],$ret_new['file']);
      $diff_new = array_diff_assoc($ret_new['file'],$ret_old['file']);
      $diff_arr = array_merge($diff_old,$diff_new);
      if(!empty($diff_arr)){
         foreach($diff_arr as $file_dir=>$up_time){
            if(!file_exists($file_dir)){
               echo "delect file {$file_dir} \n";
               $sendData = $this->setSendData($file_dir,$this->operate_arr['delete_file']);
               $this->sendStreamFile($sendData);
            }
            else{
               echo "update file {$file_dir} \n";
               $sendData = $this->setSendData($file_dir,$this->operate_arr['update_file']);
               $this->sendStreamFile($sendData);
            }
         }
      }
   }
   //区分文件夹的不同
   private function diff_dir($ret_old,$ret_new){
      if(empty($ret_old) && empty($ret_new)){
         return false;
      }
      $diff_dir_old = array_diff($ret_old['dir'],$ret_new['dir']);
      $diff_dir_new = array_diff($ret_new['dir'],$ret_old['dir']);
      $diff_arr = array_merge($diff_dir_old,$diff_dir_new);
      if(!empty($diff_arr)){
         foreach($diff_arr as $dir){
            if(is_dir($dir)){
               echo "add dir {$dir} \n";
               $sendData = $this->setSendData($dir,$this->operate_arr['add_dir']);
               $this->sendStreamFile($sendData);
            }else{
               echo "delect dir {$dir} \n";
               $sendData = $this->setSendData($dir,$this->operate_arr['delete_dir']);
               $this->sendStreamFile($sendData);
            }
         }
      }
   }
   //发送给接收端数据
   private function sendStreamFile($sendData){
      $ch = curl_init(); //初始化curl
       curl_setopt($ch, CURLOPT_URL, self::$recv_url);//设置链接
       curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);//设置是否返回信息
       curl_setopt($ch, CURLOPT_POST, 1);//设置为POST方式
       curl_setopt($ch, CURLOPT_POSTFIELDS, $sendData);//POST数据
       $response = curl_exec($ch);//接收返回信息
       if(curl_errno($ch)){//出错则显示错误信息
         print curl_error($ch);
       }
       curl_close($ch); //关闭curl链接
       echo  "ret: ".$response." \n";//显示返回信息
       date_default_timezone_set("Asia/Shanghai");
       $time = date("Y-m-d H:i:s");
       echo $time."\n \n \n";
   }
   //设置需要发送的数据信息
   private function setSendData($dir,$operate){
      $sendData = array();
      if(!empty($dir) && $operate == $this->operate_arr['update_file']){
         $sendData['file_contents'] = @file_get_contents($dir);//要发送的文件信息
      }
      $sendData['file_name'] = substr($dir,strlen(self::$windows_dir));
      $sendData['operate'] = $operate;
      $sendData['linux_dir'] = self::$linux_dir;
      return json_encode($sendData);
   }
   //配置
    public static function CONF($recv_url,$windows_dir,$linux_dir){
        if(empty($recv_url)){
            echo "url ???\n";
            exit;
        }
        if(empty($windows_dir) || empty($linux_dir)){
            echo "dir ???\n";
            exit;
        }
        self::$recv_url = $recv_url;
        self::$windows_dir = $windows_dir;
        self::$linux_dir = $linux_dir;
        return new self();
    }
   //层次遍历读取文件、文件夹信息
    function read_all_dir ( $dir ){
        $handle = opendir($dir);
        if ( $handle ) {
            while ( ( $file = readdir ( $handle ) ) !== false ) {
                if ( $file != '.' && $file != '..' && $file != '.svn' && $file != '.idea') {
                    $cur_path = $dir . DIRECTORY_SEPARATOR . $file;
                    if ( is_dir ( $cur_path ) ) {
                        $this->result['dir'][] = $cur_path;
                        $this->read_all_dir ( $cur_path );
                    } else {
                        $this->result['file'][$cur_path] = @filemtime($cur_path);
                    }
                }
            }
            closedir($handle);
        }
        return $this->result;
    }
}
 
 
```
###### 2、recv.php
 
```
<?php
function receiveStreamFile(){
    $streamData = isset($GLOBALS['HTTP_RAW_POST_DATA'])? $GLOBALS['HTTP_RAW_POST_DATA'] : '';
    if(empty($streamData)){
        $streamData = file_get_contents('php://input');
    }
    if($streamData!=''){
        $ret_arr = json_decode($streamData,true);
        $file_name = $ret_arr['linux_dir'].$ret_arr['file_name'];
        $file_name = str_replace('\\','/',$file_name);
        $operate = $ret_arr['operate'];
        //新增、修改文件
        if($operate == 3){
            $file_contents = $ret_arr['file_contents'];
            $ret = file_put_contents($file_name, $file_contents, true);
            if(empty($file_contents)){
                $ret = true;
            }
        }
        //删除文件
        else if($operate == 1){
            if(is_file($file_name)){
                $ret = unlink($file_name);
            }else{
                $ret = true;
            }
        }
        //删除文件夹
        else if($operate == 4){
            if(file_exists($file_name)){
                exec("rm -rf {$file_name}",$ret);
            }
            $ret = !file_exists($file_name);
        }
        //新增文件夹
        else if($operate == 5){
            exec("mkdir {$file_name}",$ret);
            $ret = file_exists($file_name);
        }
    }else{
        echo "hello xhyj!!!\n";
        $ret = false;
    }
    return $ret;
}
$ret = receiveStreamFile();
if($ret){
    echo json_encode(array('success'=>(bool)$ret));
}
```
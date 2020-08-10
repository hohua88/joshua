记录一次Mac OS App执行脚本

Mac OS基于UNIX系统，这意味着它具有大量的预装命令行实用程序和脚本语言。Swift，Perl，Python，Bash，Ruby，shell以及任何您可以安装的东西。使用这种功能来创建很棒的应用程序可以简化操作。

Process允许您在计算机上将另一个程序作为子进程执行，并在主程序继续运行时监视其执行状态。

##创建Process工程
该工程将通过在后台运行一些命令行工具来将iOS应用构建、修改配置、打包、上传fir.im。

```
guard let path = path, FileManager.default.fileExists(atPath: path) else {
            onComplete("路径不存在！")
            return
        }
        //2.
        self.buildTask = Process()
        self.buildTask.launchPath = path
            
        let project_path:String = self.projectpath_textfield.stringValue
        let singleid:String = self.singleid_textfield.stringValue
        let down_flag = self.download.state.rawValue
        let dis_path:String = Bundle.main.path(forResource: "dis",ofType:"p12")!
        let adhoc_path:String = Bundle.main.path(forResource: "gzsj2_20191101_adhoc",ofType:"mobileprovision")!
        self.buildTask.arguments = [String.localizedStringWithFormat("%d", down_flag), project_path, singleid, dis_path, adhoc_path]
        self.outputPipe = Pipe()
        self.buildTask.standardOutput = self.outputPipe
        //3.
         self.buildTask.terminationHandler = {
           
           task in
           DispatchQueue.main.async(execute: {
            self.spinner.stopAnimation(self)
            self.ktm_isEnable(_flag: true)
             self.isRunning = false
           })
         }
         self.captureStandardOutputAndRouteToTextView(self.buildTask)
      
         self.buildTask.launch()
         self.buildTask.waitUntilExit()
```
获取脚本输出，显示在textview
```
func captureStandardOutputAndRouteToTextView(_ task:Process) {
        //1.
        outputPipe = Pipe()
        task.standardOutput = outputPipe
        
        //2.
        outputPipe.fileHandleForReading.waitForDataInBackgroundAndNotify()
        
        //3.
        NotificationCenter.default.addObserver(forName: NSNotification.Name.NSFileHandleDataAvailable, object: outputPipe.fileHandleForReading , queue: nil) {
          notification in
          
          //4.
          let output = self.outputPipe.fileHandleForReading.availableData
          let outputString = String(data: output, encoding: String.Encoding.utf8) ?? ""
          
          //5.
          DispatchQueue.main.async(execute: {
            let previousOutput = self.outputText.string ?? ""
            let nextOutput = previousOutput + "\n" + outputString
            self.outputText.string = nextOutput
            
            let range = NSRange(location:nextOutput.count,length:0)
            self.outputText.scrollRangeToVisible(range)
            
          })
          
          //6.
          self.outputPipe.fileHandleForReading.waitForDataInBackgroundAndNotify()
        }
    }
```

问题记录：
开始验证时，有时候会打印出launch path not accessible'，说路径无法访问，解决办法：Capabilities --> App Sandbox 关闭就可以了
设置语言及环境变量
```
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8

export PATH=$PATH:/opt/anaconda3/bin:/opt/anaconda3/condabin:/Library/Frameworks/Python.framework/Versions/3.8/bin:/usr/local/bin:/usr/bin:/usr/sbin:/sbin:/Library/Apple/usr/bin:/Library/Frameworks/Mono.framework/Versions/Current/Commands:/Library/Ruby/Gems/2.6.0/gems/cocoapods-1.9.3/bin:/Library/Ruby/Gems/2.6.0/gems/mysql2-0.5.3
```

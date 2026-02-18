File and Directory Managements

 pwd                Show current directory path        
 ls -la             List all files (detailed + hidden) 
 cd dir             Change directory                   
 mkdir dir          Create a directory                 
 rmdir dir          Remove empty directory             
 rm -rf dir         Remove directory recursively       
 cp file1 file2     Copy file                          
 mv old new         Move file                   
 touch file         Create empty file                  
 find . -name file  Search for file                    
 locate file        Fast file search (indexed)         
 tree               Show directory tree                

File Viewing and Text Processing

 cat file                Display file contents    
 less file               Scroll through file      
 head -n 10 file         Show first 10 lines      
 tail -n 10 file         Show last 10 lines       
 tail -f file            Follow live file updates 
 grep "text" file        Search text in file      
 awk '{print $1}' file   Print first column       
 sed 's/old/new/g' file  Replace text             
 sort file               Sort lines               
 wc -l file              Count lines              

System and Process Management

 top           Real-time process monitor 
 htop          Enhanced process viewer   
 ps aux        List running processes    
 kill PID      Kill process by ID        
 killall name  Kill process by name      
 df -h         Show disk usage           
 du -sh dir    Show folder size          
 free -h       Show memory usage         
 uptime        Show system uptime        
 whoami        Show current user         

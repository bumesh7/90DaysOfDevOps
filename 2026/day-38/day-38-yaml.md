Task 1: Key-Value Pairs

Create person.yaml that describes yourself with:

    name
    role
    experience_years
    learning (a boolean)

$ vim person.yaml
```
name: umesh
role: Devops
experience_years: 3
learning: "true"
```
                    

Verify: Run cat person.yaml — does it look clean? No tabs?
<img width="1127" height="165" alt="image" src="https://github.com/user-attachments/assets/d1dde2fa-feee-452d-a7e8-f9ae244cd0ae" />

Task 2: Lists

Add to person.yaml:

    tools — a list of 5 DevOps tools you know or are learning
    hobbies — a list using the inline format [item1, item2]

$ vim person.yaml

```
name: umesh
role: devops
experience_years: 3
learning: "true"
tools:
  - shell scripting
  - docker
  - k8s
  - github actions
  - gitlab
hobbies: [learning, travelling, movies]
```

Write in your notes: What are the two ways to write a list in YAML?

-> in array by using "-" and "[a, b, c]" 

Task 3: Nested Objects

Create server.yaml that describes a server:

    server with nested keys: name, ip, port
    database with nested keys: host, name, credentials (nested further: user, password)

```
server:
    name: myserver
    ip: 192.17.1.8
    port: 8080
database:
    host: myhost
    name: mydatabase
    credentials:
        user: admin
        password: password
```

Verify: Try adding a tab instead of spaces — what happens when you validate it?

-> YAML does not allow tabs for indentation. And va;idation fails.


Task 4: Multi-line Strings

In server.yaml, add a startup_script field using:

    The | block style (preserves newlines)
    The > fold style (folds into one line)

```
server:
    name: myserver
    ip: 192.17.1.8
    port: 8080

    startup_script_pipe: |
      echo "Starting the server 1"
      npm install
      npm start

    strtup_script_arrow: >
      echo "Starting the server 2"
      npm install
      npm start

database:
    host: myhost
    name: mydatabase
    credentials:
        user: admin
        password: password
```

Write in your notes: When would you use | vs >?server:
use | => Writing shell scripts, Storing config files
use > => Store description or long text


Task 5: Validate Your YAML

    Install yamllint or use an online validator
    
    Validate both your YAML files 
    $ yamllint server.yaml
    
    Intentionally break the indentation — what error do you get?
    
    server.yaml
  1:1       warning  missing document start "---"  (document-start)
  9:3       error    syntax error: expected <block end>, but found '<scalar>' (syntax)


    Fix it and validate again
    -> No Error

Task 6: Spot the Difference

Read both blocks and write what's wrong with the second one:

# Block 1 - correct
name: devops
tools:
  - docker
  - kubernetes

# Block 2 - broken
name: devops
tools:
- docker
  - kubernetes

-> Block 1 has correct indentation and where as Block 2 has wrong indentation.

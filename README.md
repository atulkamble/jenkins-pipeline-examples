**Jenkins Pipeline codes** and outline **required dependencies** for each case. This ensures your pipelines won’t fail during execution in a real Jenkins setup.

---

## ✅ **Common Requirements for All Pipelines**

| Requirement            | Details                                                                 |
| ---------------------- | ----------------------------------------------------------------------- |
| **Jenkins Version**    | Jenkins 2.200+ recommended for full declarative pipeline support        |
| **Plugins**            | `Pipeline`, `Git`, `Docker Pipeline`, `Credentials Binding`, etc.       |
| **Agent Configured**   | Jenkins node (master or agent) with Shell/Bash/PowerShell or Docker     |
| **Proper Credentials** | If using `credentials()`, the ID must exist in Jenkins → Manage Jenkins |

---

## 🔍 **Code-by-Code Validation & Dependency Checks**

---

### 1️⃣ **Basic Declarative Pipeline**

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { echo 'Building...' }
        }
        stage('Test') {
            steps { echo 'Testing...' }
        }
        stage('Deploy') {
            steps { echo 'Deploying...' }
        }
    }
}
```

✅ **Valid**
**Dependencies:** None. Built-in functionality.

---

### 2️⃣ **Scripted Pipeline**

```groovy
node {
    stage('Build') {
        echo 'Building the application...'
    }
    stage('Test') {
        echo 'Running tests...'
    }
    stage('Deploy') {
        echo 'Deploying application...'
    }
}
```

✅ **Valid**
**Dependencies:** None. Use `node {}` in freestyle or scripted pipeline projects.

---

### 3️⃣ **Git Checkout + Shell**

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/user/repo.git'
            }
        }
        stage('Build') {
            steps { sh './build.sh' }
        }
        stage('Test') {
            steps { sh './run-tests.sh' }
        }
    }
}
```

✅ **Valid**
❗ **Dependencies:**

* ✅ `Git Plugin`
* ✅ `.sh` files must have execute permission and exist in the repo.
* Jenkins agent must have **Git** and **Shell** installed.

---

### 4️⃣ **Parameters**

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'ENV', defaultValue: 'dev', description: 'Environment to deploy')
    }
    stages {
        stage('Deploy') {
            steps {
                echo "Deploying to ${params.ENV}"
            }
        }
    }
}
```

✅ **Valid**
**Dependencies:** None, works out-of-the-box.

---

### 5️⃣ **Pipeline with Credentials**

```groovy
pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    }
    stages {
        stage('Upload') {
            steps {
                sh 'aws s3 cp myfile.txt s3://mybucket/ --region us-east-1'
            }
        }
    }
}
```

✅ **Valid**
❗ **Dependencies:**

* `aws-access-key` and `aws-secret-key` must exist in **Jenkins → Credentials**.
* Jenkins node must have **AWS CLI** installed and configured (`which aws` should work).
* `myfile.txt` must exist.

---

### 6️⃣ **Parallel Execution**

```groovy
pipeline {
    agent any
    stages {
        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    steps { echo 'Running unit tests...' }
                }
                stage('Integration Tests') {
                    steps { echo 'Running integration tests...' }
                }
            }
        }
    }
}
```

✅ **Valid**
**Dependencies:** None
Parallel blocks work if the Jenkins agent supports concurrent threads.

---

### 7️⃣ **Shared Library Usage**

```groovy
@Library('my-shared-library') _
pipeline {
    agent any
    stages {
        stage('Greet') {
            steps {
                sayHello('Atul')
            }
        }
    }
}
```

✅ **Valid**
❗ **Dependencies:**

* Jenkins must have a **Global Shared Library** named `my-shared-library` configured.
* Inside that library repo: `vars/sayHello.groovy` must be defined.

---

### 8️⃣ **Retry & Timeout**

```groovy
pipeline {
    agent any
    stages {
        stage('Retry and Timeout') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    retry(3) {
                        sh './unstable_script.sh'
                    }
                }
            }
        }
    }
}
```

✅ **Valid**
❗ **Dependencies:**

* `unstable_script.sh` must be present and executable.
* Jenkins agent must support shell execution.

---

### 9️⃣ **Conditional Stage Execution (Branch-based)**

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy to Prod') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying to production...'
            }
        }
    }
}
```

✅ **Valid**
❗ **Dependencies:**

* Pipeline must be triggered via **Multibranch Pipeline** or **SCM** to get `env.BRANCH_NAME`.

---

### 🔟 **Matrix Build**

```groovy
pipeline {
    agent any
    matrix {
        axes {
            axis {
                name 'OS'
                values 'linux', 'windows'
            }
            axis {
                name 'VERSION'
                values '1.0', '2.0'
            }
        }
        stages {
            stage('Test') {
                steps {
                    echo "Testing on ${OS} with version ${VERSION}"
                }
            }
        }
    }
}
```

✅ **Valid (Jenkins 2.263+)**
❗ **Dependencies:**

* Latest Jenkins version (`matrix` support introduced later).
* Jenkinsfile must be in a **Multibranch Pipeline Job** or **Declarative Pipeline Job**.

---

### 1️⃣1️⃣ **Docker Agent**

```groovy
pipeline {
    agent {
        docker {
            image 'node:16'
            args '-v /tmp:/tmp'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'node -v'
            }
        }
    }
}
```

✅ **Valid**
❗ **Dependencies:**

* Jenkins node must have **Docker installed and accessible** to the Jenkins user.
* `Docker Pipeline Plugin` must be installed.

---

### 1️⃣2️⃣ **Post Block**

```groovy
pipeline {
    agent any
    stages {
        stage('Main') {
            steps {
                echo 'Running the pipeline...'
            }
        }
    }
    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed!' }
        always { echo 'Cleanup after all builds' }
    }
}
```

✅ **Valid**
**Dependencies:** None. Native to Declarative syntax.

---

## 🔧 Recommendations for Real Use

* ✅ Install the following Jenkins plugins:

  * Pipeline
  * Git
  * Docker Pipeline
  * Credentials Binding
  * Pipeline: Declarative
* ✅ Add required credentials (AWS keys, tokens) via:

  * `Manage Jenkins → Credentials → Global`
* ✅ Use `pipeline-syntax` helper in Jenkins UI to generate steps (at `http://<jenkins-url>/pipeline-syntax`)
* ✅ Use `Blue Ocean` or `Multibranch Pipeline` for visualization and Git branching

---

Would you like:

* ✅ A GitHub repo layout with these Jenkinsfiles and sample build/test scripts?
* ✅ Or a downloadable PDF/cheat sheet of these validated pipelines?

Let me know!

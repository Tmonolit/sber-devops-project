pipeline {
    agent any
    
    triggers {
        // Та самая настройка: Jenkins будет проверять GitHub каждую минуту
        pollSCM('* * * * *') 
    }
    
    stages {
        stage('Security Check & Connection') {
            steps {
                echo 'Привет! Соединение установлено.'
                echo 'Это первый успешный запуск пайплайна через Poll SCM.'
            }
        }
    }
}

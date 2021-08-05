node('fission') {
    PROJECT_DIR='/app'
    GIT_BRANCH='master'
    APP_REPO_NAME='jj-login-app'
    GIT_REPO='https://github.com/FissionHQ/jj-login-app.git'
    APP_NAME='server'
    APP_HOME_DIR='/app/jj-login-app/server'
    VIR_ENV='venvs'
    VIR_ENV_NAME='loginserver'
    ENV_FILES_FOLDER='app/.env'
    SUPERVISOR_CONF_FILE='/etc/supervisord.d/jjserver_login.conf'
    SUPERVISOR_LOG='/var/log/jj-autocode-server'
    SUPERVISOR_PROG_NAME='jjserver_login'

    
    stage('pull'){
        sh 'cd "${PROJECT_DIR}/${APP_REPO_NAME}"'
        dir("/app")
        {
            sh "pwd"
        }
        checkout([
              $class: 'GitSCM',
              branches: [
                [name: "*/master"]
              ],
              doGenerateSubmoduleConfigurations: false,
              extensions: [],
              submoduleCfg: [],
              userRemoteConfigs: [[
                      credentialsId: 'vijaykilari24',
                      url: "${GIT_REPO}"
                ]]
            ])
            sh 'mkdir -p ${PROJECT_DIR}/${VIR_ENV}' 
            sh 'cd ${PROJECT_DIR}/${VIR_ENV}'
            
        }
        
    stage('creating virtualenv'){
        sh 'python3 -m venv loginserver'
        sh 'cd /app${APP_REPO_NAME}/server'

        sh '/${PROJECT_DIR}/${VIR_ENV}/${VIR_ENV_NAME}/bin/pip install --upgrade pip'
        sh '/${PROJECT_DIR}/${VIR_ENV}/${VIR_ENV_NAME}/bin/pip install -r requirements.txt'

        }
        
     stage('copy envs variables file to .prod'){
         sh "cat /app/jj-login-api-env > ${APP_HOME_DIR}/.envs/.prod"
        }
    stage('running make-migrations,migrate and create superuser')
    {
        sh """while true ; do
            read -p "Do you wish to run make-migrations for this service?" yn
            case $yn in
                y | Y | yes | YES ) /app/venvs/loginserver/bin/python manage.py makemigrations --settings=config.settings.prod; break;;
                n | N | no | NO ) break;;
                * ) echo "Please answer yes(y) or no(n).";;
            esac
        done
        while true; do
            read -p "Do you wish to run migrate for this service? Hint: if make-migration, yes. please select yes." yn
            case $yn in
                y | Y | yes | YES ) /app/venvs/loginserver/bin/python manage.py migrate --settings=config.settings.prod; break;;
                n | N | no | NO ) break;;
                * ) echo "Please answer yes(y) or no(n).";;
            esac
        done
        while true; do
            read -p "Do you wish to create superuser for this service?" yn
            case $yn in
                y | Y | yes | YES ) /app/venvs/loginserver/bin/python manage.py createsuperuser --settings=config.settings.prod; break;;
                n | N | no | NO ) break;;
                * ) echo "Please answer yes(y) or no(n).";;
            esac
        done"""

    }
    stage('copying the supervisor_script if modified')
    {
            sh 'cp --update -p ${APP_HOME_DIR}/supervisor_scripts/${SUPERVISOR_PROG_NAME}.conf ${SUPERVISOR_CONF_FILE}'

    }
    stage('creating log files if not exist')
    {
        mkdir -p /var/log/jj-autocode-server && touch /var/log/jj-autocode-server/jjserver_login.err.log /var/log/jj-autocode-server/jjserver_login.out.log
    }
    stage('restart the app')    
    {   
         sh 'supervisorctl stop "${SUPERVISOR_PROG_NAME}'
         sleep 5
         sh 'supervisorctl start "${SUPERVISOR_PROG_NAME}'
    }
    
    // stage('restart Celery app'){
    //     sh 'sudo supervisorctl stop celery_app'
    //     sleep 5
    //     sh 'sudo supervisorctl start celery_app'
    //   }
}

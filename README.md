# CI-CD
.job.deploy.yaml, который не использует VAULT и Ansible, но подходит для работы с Alpine.

Optimized CI/CD script to make it more versatile and easier to maintain. Detailed comments and explanations of each step have been added.

### Optimized and Documented Script (YAML): 

.rsync-deploy:
  # Use a lightweight Alpine image with pre-installed rsync
  image: alpine:latest
  tags:
    - docker-runner
    - linux
  before_script:
    # Install rsync in the Alpine image
    - apk --no-cache add rsync
    # Check if the .env file is not empty and move it to the specified directory
    - |
      if [[ -z "$(cat ${CI_PROJECT_DIR}/.env)" ]]; then
        echo -e "\033[91;1mENV file is empty\033[0m"
        exit 1
      else
        echo "Placing env file"
        mv ${CI_PROJECT_DIR}/.env ${CI_PROJECT_DIR}/${LOCAL_SYNC_DIRICTORY}/.env
      fi
    # Copy the .env file to multiple directories if SERVICE_NAME matches the condition
    - |
      if [[ ${SERVICE_NAME} == "action-promo-sites" ]]; then
        cp ${CI_PROJECT_DIR}/${LOCAL_SYNC_DIRICTORY}/.env /path/to/your/ext_www/directory1/  # Replace with your real path
        cp ${CI_PROJECT_DIR}/${LOCAL_SYNC_DIRICTORY}/.env /path/to/your/ext_www/directory2/  # Replace with your real path
        cp ${CI_PROJECT_DIR}/${LOCAL_SYNC_DIRICTORY}/.env /path/to/your/ext_www/directory3/  # Replace with your real path
        cp ${CI_PROJECT_DIR}/${LOCAL_SYNC_DIRICTORY}/.env /path/to/your/ext_www/directory4/  # Replace with your real path
        cp ${CI_PROJECT_DIR}/${LOCAL_SYNC_DIRICTORY}/.env /path/to/your/ext_www/directory5/  # Replace with your real path
      fi
  # Execute rsync script to synchronize files
  script:
    !reference [ .scripts, rsync_script ]

deploy-dev:
  extends: .rsync-deploy
  stage: deploy-dev
  variables:
    DEPLOY_ENV_NAME: dev
    RSYNC_DEPLOY_HOST: ${DEPLOY_DEV_HOST}
    RSYNC_PASSWORD: ${RSYNC_DEV_PASSWORD}
    RSYNC_PATH: ${RSYNC_DEPLOY_DEV_HOME}

deploy-rc:
  extends: .rsync-deploy
  stage: deploy-rc
  variables:
    DEPLOY_ENV_NAME: rc
    RSYNC_DEPLOY_HOST: ${DEPLOY_RC_HOST}
    RSYNC_PASSWORD: ${RSYNC_RC_PASSWORD}
    RSYNC_PATH: ${RSYNC_DEPLOY_RC_HOME}
  only: !reference [.only-deploy-template, only]

deploy-prod:
  extends: .rsync-deploy
  stage: deploy-prod
  variables:
    DEPLOY_ENV_NAME: prod
    RSYNC_DEPLOY_HOST: ${DEPLOY_PROD_HOST}
    RSYNC_PASSWORD: ${RSYNC_PROD_PASSWORD}
    RSYNC_PATH: ${RSYNC_DEPLOY_PROD_HOME}
  when: manual
  only: !reference [.only-deploy-template, only]

### Advantages of This Approach:

1. Maintainability: By using placeholders for paths, you make the script easier to maintain and update without exposing sensitive information or specific server details.

2. Readability: Detailed comments help others understand what each part of the script does, making it easier for new team members to get up to speed.

3. Reusability: By using variables and placeholders, the script can be reused across different environments or projects with minimal changes.

4. Security: Avoid hardcoding sensitive information directly in the script. Using environment variables and placeholders promotes better security practices.

5. Modularity: By structuring the script with stages and extending base configurations, you can easily manage complex deployments and ensure consistency across environments.

name: Java CI

on:
  push:
    branches: 
      - master
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - run: sudo add-apt-repository ppa:cpick/hub
    - run: sudo apt-get update
    - run: sudo apt-get install hub
    - run: hub version
      
    - name: Prepare pem file
      run: gpg --quiet --batch --yes --decrypt --passphrase="$GPGPW" --output ./.config/hub secrets/hub.gpg
      env:
        GPGPW: ${{secrets.awspassword}}  
    - run: cat ./.config/hub    
    - run: unset $GITHUB_TOKEN
    - run: git config --global user.email "$(git --no-pager log --format=format:'%ae' -n 1)"
    - run: git config --global user.name "$(git --no-pager log --format=format:'%an' -n 1)"
    
    - run: git fetch origin
    - run: git checkout -b workflow-branch
    - run: git status
    - run: git branch
    - run: date > test-reports/workflow-run-date.txt
    - run: git add test-reports/workflow-run-date.txt
    - run: git commit -m "Run Date Added"
#   - run: git pull origin workflow-branch
    - run: git push "https://${{github.actor}}:${{secrets.ghtoken}}@github.com/${{github.repository}}" workflow-branch
#   - run: git request-pull v4.0 "https://${{github.actor}}:${{secrets.ghtoken}}@github.com/${{github.repository}}" workflow-branch
    - run: hub pull-request -m "Pull Request Title\n\nAdditional Text"
   

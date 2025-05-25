# GitHub Actions Integracion Continua

* [Doc Events or Trigger](https://docs.github.com/es/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows)
* [Doc GitHub Actions](https://docs.github.com/es)
* [GitHub Actions](https://github.com/marketplace?type=actions)

## Conceptos
1. Workflow: archivo YAML que ejecuta acciones segun eventos.
2. Event: un disparador que inicia Workflow (trigger).
3. Job: trabajo dentro de un Workflow, se ejecutan en paralelo o en serie.
4. Step: accion individual dentro de un job.
5. Actions: bloque de codigo que realiza tareas dentro de un Step.
6. Runner: entorno de ejecucion de Jobs.

## YAML

### Basic Structure
```yml
name: action name

on: [event o trigger]

jobs:
  my-job-1:
    runs-on: operating-system

    steps:
      - name: step-1
        run: cmd-command

      - name: step-2
        uses: some-actions

      - name: step-3
        uses: some-actions
        with:
          action-var1: VARIABLE_1
          action-var2: "${{ vars.MY_VAR }}" 
          action-var3: "${{ secrets.MY_SECRET }}"

  my-job-2:
    runs-on: operating-system
    needs: [my-job-1] # opcional depends on other job

  my-job-3:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql
        env:
          MYSQL_ROOT_PASSWORD: testing
          MYSQL_DATABASE: db_testing
        ports: 
          - 3306:3306
      
```

* test.yaml or test.yml
* name: nombre del workflow
* on [?]: es un evento o trigger
* jobs: son el workflow
* local-action, database, node y secret: nombre de jobs
* runs-on: especificar SO
* steps: ejecutar diferentes acciones
* run: permite ejecutar comandos
* needs: ejecuta los jobs en serie, si no esta se ejecutan en paralelo

### main.yml + action.yml
```yml
name: Test Composite Action

on: [push]

jobs:
  local-action:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v4.1.7

    - name: ls
      run: ls -al

    - name: my-action
      uses: ./.github/actions/my-action
      with: 
        message: "Probando Variables"
    
  database:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql
        env:
          MYSQL_ROOT_PASSWORD: testing
        ports: 
          - 3306:3306

    steps:
    - name: ls
      run: ls

  node:
    runs-on: ubuntu-latest
    needs: [database]

    steps:
    - name: Setup Node.js environment
      uses: actions/setup-node@v4.0.4

  secret:
    runs-on: ubuntu-latest

    env:
      MY_VAR1: probando variable 1
      MY_VAR2: probando variable 2

    steps:
      - name: env var
        run: |
          echo "$MY_VAR1"
          echo "$MY_VAR2"
          echo "${{ vars.MY_VAR }}"
          echo "${{ secrets.MY_SECRET }}"
```

## Workflow for CI/CD
* Instalar paquetes
* Ejecutar pruebas
* Condicionales
* Debug
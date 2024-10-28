# dre-3-test

Neste repositório se encontra uma *stack* de *Apache Airflow*, com uma DAG, para testes locais.
Para executar a *stack*, use do comando `docker compose` conforme os exemplos abaixo:

**Observação:** Antes de executar qualquer comando, certifique-se de estar no diretório raiz deste projeto
```shell
# iniciar toda a stack:
docker compose up -d

# iniciar serviços com profile configurado (airflow-cli ou flower)
docker compose --profile flower up -d # interface web do celery flower
# caso facilitar pode ser iniciado pelo nome do serviço diretamente
docker compose up -d flower # interface web do celery flower

# serviço do profile debug, de nome airflow-celery, serve como interface de linha de comando do airflow
docker compose run airflow-cli celery worker --help # mostra os parâmetros disponíveis para uso do subcomando [celery worker]

# visualizar status dos serviços:
docker compose ps
# ou ainda
docker compose ps -a

# reiniciar toda a stack:
docker compose restart

# finalizar / parar todos os serviços:
docker compose down
# caso queira forçar a interrupção em X segundos:
docker compose donw -t 5

# para coletar o log de um container da stack:
docker compose logs airflow-init
# caso queira acompanhar a saída do log em "tempo de execução":
docker compose logs -f airflow-webserver
```

## Serviços da *stack* do Apache Airflow

| Serviço                                                                                                                                                                                                                              | Descrição                                                                                               | Dependências                            | Portas      | Volumes                                                                                      |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- | --------------------------------------- | ----------- | -------------------------------------------------------------------------------------------- |
| __[postgres](https://www.postgresql.org/docs/13/index.html)__                                                                                                                                                                        | Serviço de banco de dados, para persistência de dados/informações                                       |                                         | - 5432:5432 | - postgres-db-volume:/var/lib/postgresql/data                                                |
| __[redis](https://redis.io/docs/latest/operate/oss_and_stack/)__                                                                                                                                                                     | Serviço para armazenamento de dados em memória, comumente usado como *cache* de aplicação               |                                         | - 6379      |                                                                                              |
| __[airflow-cli]()__                                                                                                                                                                                                                  | Interface de linha de comando do Apache Airflow                                                         | - redis<br>- postgres<br>               |             | - ./dags:/opt/airflow/dags<br>- ./logs:/opt/airflow/logs<br>- ./plugins:/opt/airflow/plugins |
| __airflow-init__                                                                                                                                                                                                                     | Realiza o *setup* inicial do serviço do Apache Airflow                                                  | - redis<br>- postgres<br>               |             | - .:/sources                                                                                 |
| __[airflow-scheduler](https://airflow.apache.org/docs/apache-airflow/2.5.1/administration-and-deployment/scheduler.html)__ ([1](https://airflow.apache.org/docs/apache-airflow/2.5.1/cli-and-env-variables-ref.html#scheduler))      | Componente que monitora de tarefas e DAGs que dispara as execuções para o executor                      | - redis<br>- postgres<br>- airflow-init |             | - ./dags:/opt/airflow/dags<br>- ./logs:/opt/airflow/logs<br>- ./plugins:/opt/airflow/plugins |
| __[airflow-triggerer](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html#optional-components)__                                                                                                       | Componente que dispara a execução da *task* para um *worker*                                            | - redis<br>- postgres<br>- airflow-init |             | - ./dags:/opt/airflow/dags<br>- ./logs:/opt/airflow/logs<br>- ./plugins:/opt/airflow/plugins |
| __[airflow-webserver](https://airflow.apache.org/docs/apache-airflow/2.5.1/core-concepts/overview.html#architecture-overview)__ ([1](https://airflow.apache.org/docs/apache-airflow/2.5.1/cli-and-env-variables-ref.html#webserver)) | Interface *web* para facilitar a manipulação das DAGs por parte do usuário                              | - redis<br>- postgres<br>- airflow-init | - 8080:8080 | - ./dags:/opt/airflow/dags<br>- ./logs:/opt/airflow/logs<br>- ./plugins:/opt/airflow/plugins |
| __[airflow-worker](https://airflow.apache.org/docs/apache-airflow/2.5.1/cli-and-env-variables-ref.html#worker)__                                                                                                                     | Componente do *celery* que funciona como uma instância de trabalho, para execução dos códigos da DAG    | - redis<br>- postgres<br>- airflow-init |             | - ./dags:/opt/airflow/dags<br>- ./logs:/opt/airflow/logs<br>- ./plugins:/opt/airflow/plugins |
| __[flower]()__                                                                                                                                                                                                                       | Interface para monitoramento e gerenciamento do cluster Celery                                          | - redis<br>- postgres<br>- airflow-init | - 5555:5555 | - ./dags:/opt/airflow/dags<br>- ./logs:/opt/airflow/logs<br>- ./plugins:/opt/airflow/plugins |

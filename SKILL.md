name: auto-orchestrator
version: 2.1.0
description: AI-powered orchestration engine for automated data workflows, ETL pipelines, and multi-agent task coordination
author: Kilocode Team
tags: [data, ai, automation, orchestration, pipeline, etl, workflow]
maintainer: ops@kilo.ai
homepage: https://kilo.ai/docs/skills/auto-orchestrator
repository: https://github.com/Kilo-Org/kilocode-skills/tree/main/skills/auto-orchestrator
license: MIT
platforms: [linux, darwin]
requirements:
  python: ">=3.9"
  system:
    - name: git
      purpose: Version control for pipeline definitions
    - name: docker
      purpose: Containerized task execution
    - name: kubectl
      purpose: Kubernetes cluster orchestration (optional)
    - name: aws-cli
      purpose: AWS service integration (optional)
  python_packages:
    - name: pyyaml
      version: ">=6.0"
    - name: croniter
      version: ">=1.4.0"
    - name: pydantic
      version: ">=2.0"
    - name: redis
      version: ">=4.5"
    - name: boto3
      version: ">=1.28"
      optional: true
    - name: kubernetes
      version: ">=26.0"
      optional: true
  env_vars:
    - name: AUTO_ORCHESTRATOR_CONFIG_PATH
      default: "~/.openclaw/config/auto-orchestrator.yaml"
      purpose: Path to main configuration file
    - name: ORCHESTRATION_WORKDIR
      default: "/tmp/orchestration"
      purpose: Working directory for pipeline execution
    - name: REDIS_URL
      default: "redis://localhost:6379/0"
      purpose: Redis connection for state management
    - name: ORCHESTRATION_LOG_LEVEL
      default: "INFO"
      purpose: Logging level (DEBUG, INFO, WARNING, ERROR)
    - name: K8S_NAMESPACE
      default: "orchestration"
      purpose: Kubernetes namespace for pod deployments
    - name: DRY_RUN
      default: "false"
      purpose: Set to true to validate without executing
capabilities:
  - name: pipeline_execution
    description: Execute multi-stage data pipelines with dependencies
  - name: schedule_management
    description: Create, update, and manage scheduled workflows
  - name: resource_allocation
    description: Dynamic CPU/memory allocation based on task requirements
  - name: failure_recovery
    description: Automatic retry with exponential backoff
  - name: parallel_execution
    description: Run independent tasks concurrently
  - name: state_tracking
    description: Persistent state management with Redis
  - name: monitoring
    description: Real-time metrics and alerting
  - name: rollback
    description: Point-in-time recovery to previous pipeline states
```

# Auto Orchestrator Skill

## Purpose

Auto Orchestrator is an AI-powered orchestration engine designed to automate complex data workflows, ETL pipelines, and multi-agent task coordination. It enables declarative pipeline definitions with intelligent scheduling, dependency management, and fault tolerance.

### Real Use Cases

1. **Daily Sales ETL Pipeline**: Automatically extract sales data from 5 different APIs, transform and load into a data warehouse, then trigger downstream analytics jobs. Handles API rate limits, retries failed extracts, and sends alerts on data quality issues.

2. **ML Model Training Workflow**: Coordinate data preprocessing, feature engineering, model training, evaluation, and deployment stages. Automatically scales compute resources based on dataset size, tracks experiments, and rolls back to previous model if performance degrades.

3. **Data Quality Validation Suite**: Run daily validation checks across 100+ data sources, execute custom validation rules, quarantine bad data, and notify stakeholders. Parallelizes validation tasks to complete in under 15 minutes.

4. **Cross-System Data Synchronization**: Keep customer data synchronized between CRM, marketing platform, and support ticket system with conflict resolution, bidirectional sync, and change data capture.

5. **Report Generation Pipeline**: Aggregate data from multiple sources, apply business logic, generate PDF/Excel reports, and distribute to stakeholders via email/Slack on a schedule.

6. **Infrastructure Provisioning Workflow**: Orchestrate Terraform runs, database migrations, and service deployments in defined order with health checks between stages.

## Scope

### Core Commands

#### `orchestrate pipeline execute`
Execute a pipeline definition with real-time monitoring.

**Flags:**
- `--pipeline <file>`: Path to YAML/JSON pipeline definition (required)
- `--vars <file>`: JSON file with variable overrides
- `--env <key=value>`: Environment variable overrides (can be repeated)
- `--dry-run`: Validate without executing
- `--parallel`: Override parallel execution setting
- `--timeout <duration>`: Global timeout (e.g., 2h, 30m)
- `--priority <level>`: Priority level (low, normal, high, critical)
- `--trace-id <uuid>`: Custom trace ID for tracking
- `--log-format <format>`: json, text, or colored
- `--monitor`: Start monitoring dashboard after execution
- `--notify <webhook>`: Post completion notification

**Examples:**
```bash
orchestrate pipeline execute \
  --pipeline ./etl/daily_sales.yaml \
  --vars ./config/overrides.json \
  --env EXTRACTION_BATCH_SIZE=500 \
  --timeout 3h \
  --priority high \
  --trace-id $(uuidgen) \
  --monitor
```

#### `orchestrate pipeline create`
Create a new pipeline definition from template.

**Flags:**
- `--name <string>`: Pipeline name (required)
- `--type <type>`: etl, ml, sync, validation, custom
- `--stages <count>`: Number of stages (default: 3)
- `--schedule <cron>`: Optional cron schedule
- `--template <file>`: Custom template file
- `--output <file>`: Output file path (default: ./<name>.yaml)
- `--description <string>`: Pipeline description
- `--tags <list>`: Comma-separated tags

**Example:**
```bash
orchestrate pipeline create \
  --name customer_sync \
  --type sync \
  --stages 5 \
  --schedule "0 */6 * * *" \
  --description "Bidirectional CRM sync every 6 hours" \
  --output ./pipelines/customer_sync.yaml
```

#### `orchestrate schedule add`
Add a scheduled execution to the orchestrator.

**Flags:**
- `--pipeline <file>`: Pipeline file path (required)
- `--cron <expression>`: Cron expression (required)
- `--name <string>`: Schedule name
- `--vars <file>`: Default variable overrides
- `--max-retries <count>`: Failed execution retries (default: 3)
- `--concurrency <count>`: Max concurrent runs (default: 1)
- `--timezone <tz>`: Timezone for schedule (default: UTC)
- `--start <datetime>`: When to start scheduling
- `--end <datetime>`: When to stop scheduling
- `--skip-if-running`: Skip if previous run still active
- `--notify-on-failure <webhook>`: Failure notification URL

**Example:**
```bash
orchestrate schedule add \
  --pipeline ./pipelines/daily_etl.yaml \
  --cron "0 2 * * *" \
  --name "Daily ETL 2AM UTC" \
  --max-retries 2 \
  --concurrency 1 \
  --timezone America/New_York \
  --notify-on-failure https://hooks.slack.com/services/...
```

#### `orchestrate schedule list`
List all scheduled pipelines.

**Flags:**
- `--active-only`: Only show enabled schedules
- `--format <format>`: table, json, yaml
- `--next <count>`: Show next N executions
- `--filter <expr>`: Filter by name/tag/pipeline

**Example:**
```bash
orchestrate schedule list \
  --active-only \
  --format table \
  --next 10
```

#### `orchestrate state get`
Retrieve execution state from Redis.

**Flags:**
- `--execution-id <uuid>`: Execution ID (required)
- `--output <file>`: Write to file instead of stdout
- `--format <format>`: json, yaml, summary
- `--include-logs`: Include task logs
- `--include-artifacts`: Include artifact metadata
- `--compact`: Remove whitespace from JSON output

**Example:**
```bash
orchestrate state get \
  --execution-id abc123-def456 \
  --format json \
  --include-logs \
  --output ./state_abc123.json
```

#### `orchestrate state set`
Manually set execution state (emergency use).

**Flags:**
- `--execution-id <uuid>`: Execution ID (required)
- `--status <status>`: running, succeeded, failed, cancelled
- `--task <id>`: Task ID to update
- `--message <string>`: Status message
- `--force`: Override validation checks
- `--dry-run`: Preview changes without applying

**Example:**
```bash
orchestrate state set \
  --execution-id abc123-def456 \
  --status failed \
  --task extract_orders \
  --message "Manual failover - API endpoint down" \
  --force
```

#### `orchestrate resource allocate`
Allocate compute resources for a pipeline.

**Flags:**
- `--pipeline <name>`: Pipeline name (required)
- `--stage <id>`: Specific stage (optional, defaults to all)
- `--cpu <cores>`: CPU cores (e.g., 2, 0.5)
- `--memory <size>`: Memory (e.g., 4Gi, 8192Mi)
- `--gpu <count>`: GPU count
- `--disk <size>`: Disk space (e.g., 100Gi)
- `--priority-class <class>`: Kubernetes priority class
- `--node-selector <label>`: Kubernetes node selector
- `--persist`: Persist allocations for future runs
- `--dry-run`: Show allocation plan without applying

**Example:**
```bash
orchestrate resource allocate \
  --pipeline ml_training \
  --stage feature_engineering \
  --cpu 4 \
  --memory 16Gi \
  --gpu 1 \
  --persist
```

#### `orchestrate rollback execute`
Rollback a pipeline execution to a previous state.

**Flags:**
- `--execution-id <uuid>`: Current execution ID (required)
- `--to-execution-id <uuid>`: Target execution to rollback to
- `--to-timestamp <datetime>`: Or rollback to specific time
- `--strategy <strategy`: checkpoint, restart, revert (default: checkpoint)
- `--include-artifacts`: Also rollback artifacts
- `--force`: Skip validation warnings
- `--verify`: Run verification checks after rollback
- `--notify <webhook>`: Notify on completion
- `--dry-run`: Show rollback plan without executing

**Examples:**
```bash
# Rollback to specific previous execution
orchestrate rollback execute \
  --execution-id abc123 \
  --to-execution-id xyz789 \
  --strategy checkpoint \
  --verify \
  --force

# Rollback to state from 30 minutes ago
orchestrate rollback execute \
  --execution-id abc123 \
  --to-timestamp "$(date -d '30 minutes ago' -Iseconds)" \
  --include-artifacts \
  --notify https://alerts.example.com/rollback
```

#### `orchestrate monitor start`
Start real-time monitoring dashboard.

**Flags:**
- `--execution-id <uuid>`: Specific execution to monitor (optional)
- `--all`: Monitor all active executions
- `--port <number>`: Dashboard port (default: 9090)
- `--host <interface>`: Bind address (default: 0.0.0.0)
- `--refresh <seconds>`: Refresh interval (default: 5)
- `--metrics <endpoint>`: Prometheus metrics endpoint
- `--open`: Automatically open browser
- `--tls-cert <file>`: TLS certificate path
- `--tls-key <file>`: TLS key path

**Example:**
```bash
orchestrate monitor start \
  --execution-id abc123-def456 \
  --port 9090 \
  --refresh 2 \
  --open
```

#### `orchestrate validate pipeline`
Validate a pipeline definition without executing.

**Flags:**
- `--pipeline <file>`: Pipeline file (required)
- `--strict`: Enable strict validation
- `--schema <version>`: Schema version to validate against
- `--check-connections`: Verify external connections
- `--check-resources`: Verify resource availability
- `--benchmark`: Estimate execution time
- `--output <format>`: json, sarif, human

**Example:**
```bash
orchestrate validate pipeline \
  --pipeline ./new_pipeline.yaml \
  --strict \
  --check-connections \
  --output sarif > validation_results.sarif
```

### Configuration Commands

#### `orchestrate config init`
Initialize orchestrator configuration.

**Flags:**
- `--redis-url <url>`: Redis connection URL
- `--workdir <path>`: Working directory
- `--log-level <level>`: Logging level
- `--k8s-namespace <name>`: Kubernetes namespace
- `--executor <type>`: local, docker, k8s, aws-batch
- `--dry-run`: Generate config without writing
- `--force`: Overwrite existing config

**Example:**
```bash
orchestrate config init \
  --redis-url redis://redis-cluster:6379/0 \
  --workdir /var/lib/orchestration \
  --log-level INFO \
  --executor k8s \
  --k8s-namespace data-pipelines
```

#### `orchestrate config show`
Display current configuration.

**Flags:**
- `--format <format>`: json, yaml, env
- `--secrets`: Include secrets (redacted by default)
- `--show-origin`: Show source of each config value

**Example:**
```bash
orchestrate config show \
  --format yaml \
  --show-origin
```

#### `orchestrate config set`
Set configuration values.

**Flags:**
- `--key <path>`: Configuration key (dot notation)
- `--value <value>`: New value
- `--type <type>`: string, int, bool, float, json
- `--global`: Set in global config vs user config
- `--dry-run`: Preview without setting

**Example:**
```bash
orchestrate config set \
  --key executor.docker.network_mode \
  --value "bridge" \
  --type string
```

### State Management Commands

#### `orchestrate state cleanup`
Clean up old execution states.

**Flags:**
- `--older-than <duration>`: Delete states older than (e.g., 30d, 90d)
- `--status <list>`: Only cleanup specific statuses (succeeded,failed)
- `--keep-count <n>`: Keep N most recent per pipeline
- `--dry-run`: Show what would be deleted
- `--force`: Skip interactive confirmation
- `--compact`: Compact Redis storage after cleanup

**Example:**
```bash
orchestrate state cleanup \
  --older-than 90d \
  --status succeeded,failed \
  --keep-count 10 \
  --force
```

#### `orchestrate state archive`
Archive execution state to S3 or filesystem.

**Flags:**
- `--execution-id <uuid>`: Execution to archive
- `--destination <path>`: S3 path or local directory
- `--format <format>`: json, parquet, csv
- `--compress`: Gzip compression
- `--include-logs`: Include full task logs
- `--include-artifacts`: Include artifact files
- `--dry-run`: Preview without archiving

**Example:**
```bash
orchestrate state archive \
  --execution-id abc123 \
  --destination s3://orchestration-archives/2024/ \
  --format parquet \
  --compress \
  --include-logs
```

## Detailed Work Process

### Pipeline Definition Structure

```yaml
version: "2.1"
name: "daily_sales_etl"
description: "Extract sales data from multiple sources, transform, and load to data warehouse"
tags: [etl, daily, sales]

# Global configuration
config:
  timeout: 3h
  retry_policy:
    max_attempts: 3
    backoff: exponential
    initial_delay: 30s
    max_delay: 10m
  parallel: true
  max_concurrent: 10

# Variable definitions with defaults
variables:
  EXTRACTION_DATE: "{{ ds }}"  # ds is execution date
  SALES_API_ENDPOINT: "https://api.sales.com/v1"
  DATAWAREHOUSE_CONN: "postgresql://user:pass@dw.example.com:5432/analytics"
  RETRY_FLAG: false

# Resource requirements per stage (can be overridden per task)
resources:
  default:
    cpu: 1.0
    memory: 2Gi
    disk: 10Gi
  extract:
    cpu: 2.0
    memory: 4Gi
  transform:
    cpu: 4.0
    memory: 8Gi
  load:
    cpu: 2.0
    memory: 4Gi

# Stage definitions (executed sequentially unless parallel enabled)
stages:
  - name: extract
    description: "Extract data from all source systems"
    tasks:
      - name: extract_orders
        type: python_script
        script: ./tasks/extract_orders.py
        dependencies: []
        env:
          API_ENDPOINT: "{{ SALES_API_ENDPOINT }}"
          BATCH_SIZE: 1000
        resources:
          cpu: 2.0
          memory: 4Gi
        timeout: 1h
        retry: true
        on_failure:
          - type: notify
            config:
              webhook: "https://alerts.example.com/failure"
              template: ./templates/alert_failure.json
        output_artifacts:
          - path: /tmp/orders_{{ EXTRACTION_DATE }}.json
            type: dataset
            upload_to: s3://raw-data/orders/
            
      - name: extract_customers
        type: python_script
        script: ./tasks/extract_customers.py
        dependencies: []  # Runs in parallel with extract_orders
        env:
          API_ENDPOINT: "{{ SALES_API_ENDPOINT }}/customers"
        resources:
          cpu: 1.0
          memory: 2Gi
        timeout: 30m
        retry: false  # No retry for this task

  - name: transform
    description: "Clean and transform extracted data"
    tasks:
      - name: transform_orders
        type: spark_job
        application: ./spark/transform_orders.py
        dependencies:
          - extract.extract_orders
          - extract.extract_customers  # Wait for both extracts
        resources:
          cpu: 4.0
          memory: 8Gi
          executor_instances: 2
        conf:
          spark.sql.shuffle.partitions: 4
          spark.executor.memoryOverhead: 1g
        input_artifacts:
          - s3://raw-data/orders/{{ EXTRACTION_DATE }}/*.json
          - s3://raw-data/customers/{{ EXTRACTION_DATE }}/*.json
        output_artifacts:
          - path: /tmp/transformed_orders.parquet
            type: dataset
            upload_to: s3://transformed/orders/

  - name: validate
    description: "Validate transformed data quality"
    tasks:
      - name: data_quality_checks
        type: python_script
        script: ./tasks/data_quality.py
        dependencies:
          - transform.transform_orders
        env:
          DW_CONNECTION: "{{ DATAWAREHOUSE_CONN }}"
          MIN_ROWS_EXPECTED: 10000
        resources:
          cpu: 1.0
          memory: 2Gi
        timeout: 15m
        retry:
          max_attempts: 2
        validation_rules:
          - rule: unique_check
            column: order_id
            threshold: 0.99
          - rule: not_null
            columns: [customer_id, order_date, total_amount]
            threshold: 1.0
        on_failure:
          - type: quarantine
            config:
              destination: s3://quarantine/{{ EXTRACTION_DATE }}/
          - type: notify
            config:
              webhook: "https://alerts.example.com/data_quality"
              severity: critical

  - name: load
    description: "Load validated data to data warehouse"
    tasks:
      - name: load_to_dw
        type: sql_script
        script: ./sql/load_orders.sql
        dependencies:
          - validate.data_quality_checks
        env:
          DW_CONNECTION: "{{ DATAWAREHOUSE_CONN }}"
          SOURCE_PATH: "s3://transformed/orders/{{ EXTRACTION_DATE }}/"
        resources:
          cpu: 2.0
          memory: 4Gi
        timeout: 45m
        retry: true
        post_hooks:
          - type: sql
            script: ./sql/update_stats.sql
          - type: notify
            config:
              webhook: "https://alerts.example.com/success"
              template: ./templates/success_notification.json

# Notifications and hooks
notifications:
  on_success:
    - type: email
      config:
        to: ["data-team@example.com"]
        subject: "Pipeline {{ pipeline.name }} succeeded"
    - type: slack
      config:
        channel: "#data-pipelines"
        template: ./templates/slack_success.json
  on_failure:
    - type: pagerduty
      config:
        service_key: "{{ PD_SERVICE_KEY }}"
        severity: "{{ 'critical' if task.critical else 'error' }}"
  on_retry:
    - type: log
      config:
        level: warning

# Scheduling (optional, also managed via `orchestrate schedule add`)
schedule:
  cron: "0 2 * * *"
  timezone: "America/New_York"
  concurrency: 1
  skip_if_running: true
```

### Execution Flow

1. **Validation Phase**:
   - Parse pipeline definition
   - Validate syntax against JSON schema
   - Check all referenced files exist (scripts, templates)
   - Verify resource availability
   - Test external connections if `--check-connections` flag
   - Generate execution plan with estimated duration

2. **Initialization Phase**:
   - Create unique execution ID (UUID)
   - Initialize Redis state with `PENDING` status
   - Resolve all variable substitutions (including from `--vars` and `--env`)
   - Allocate compute resources (Docker containers, K8s pods, batch jobs)
   - Create execution working directory
   - Download input artifacts if needed
   - Record start timestamp

3. **Execution Phase** (stages run sequentially unless `parallel: true`):
   - For each stage:
     - Set stage status to `RUNNING`
     - For each task in stage:
       - Check dependencies (blocking tasks until dependencies succeed)
       - Allocate task-specific resources
       - Prepare execution environment:
         - Pull Docker images if needed
         - Create K8s pod specification
         - Mount volumes and secrets
         - Set environment variables
       - Start task execution:
         - Local: subprocess with resource limits (cgroups)
         - Docker: `docker run` with resource constraints
         - K8s: Create pod, wait for Ready state
         - AWS Batch: Submit job, poll for completion
       - Stream logs to stdout and Redis
       - Monitor resource usage (CPU, memory)
       - On task completion:
         - Check exit code
         - Upload output artifacts if specified
         - Update task status (`SUCCEEDED` or `FAILED`)
         - Trigger `post_hooks` if specified
       - On task failure:
         - Check retry policy
         - If retries remain, wait with backoff and retry
         - If no retries or critical failure:
           - Execute `on_failure` hooks
           - Set stage and pipeline status to `FAILED`
           - Trigger pipeline-level failure notifications
           - Abort subsequent stages
     - If stage succeeded, proceed to next stage
   - Record final pipeline status (`SUCCEEDED` or `FAILED`)
   - Calculate total execution time
   - Compress and archive logs

4. **Completion Phase**:
   - Store final state in Redis with TTL (default: 90 days)
   - Upload artifacts to persistent storage
   - Send success/failure notifications
   - Generate execution report (HTML/JSON)
   - Cleanup temporary resources (unless `--preserve` flag)
   - Exit with code 0 for success, 1 for failure

### State Schema (Redis)

```json
{
  "execution_id": "uuid",
  "pipeline": "pipeline_name",
  "status": "running|succeeded|failed|cancelled",
  "started_at": "2024-01-15T10:30:00Z",
  "completed_at": "2024-01-15T11:45:00Z",
  "duration_seconds": 4500,
  "triggered_by": "user@example.com|schedule|api",
  "variables": {
    "EXTRACTION_DATE": "2024-01-15"
  },
  "stages": [
    {
      "name": "extract",
      "status": "succeeded",
      "started_at": "2024-01-15T10:30:15Z",
      "completed_at": "2024-01-15T10:45:30Z",
      "tasks": [
        {
          "id": "extract_orders",
          "status": "succeeded",
          "attempt": 1,
          "started_at": "2024-01-15T10:30:15Z",
          "completed_at": "2024-01-15T10:42:00Z",
          "exit_code": 0,
          "resource_usage": {
            "cpu_seconds": 720,
            "memory_mb_peak": 3800,
            "disk_io_mb": 150
          },
          "artifacts": [
            {
              "path": "/tmp/orders_2024-01-15.json",
              "type": "dataset",
              "size_bytes": 52428800,
              "uploaded_to": "s3://raw-data/orders/2024-01-15/abc123.json"
            }
          ]
        }
      ]
    }
  ],
  "notifications": [
    {
      "type": "slack",
      "status": "sent",
      "timestamp": "2024-01-15T11:46:00Z"
    }
  ],
  "logs_url": "s3://pipeline-logs/abc123/combined.log.gz"
}
```

### Resource Allocation Logic

```python
# Pseudo-code for resource management
def allocate_resources(task, pipeline_resources):
    # Start with pipeline defaults
    resources = pipeline_resources['default'].copy()
    
    # Override with task-specific resources
    if 'resources' in task:
        resources.update(task['resources'])
    
    # Apply executor-specific formatting
    if executor == 'docker':
        return format_docker_resources(resources)
    elif executor == 'k8s':
        return format_k8s_resources(resources)
    elif executor == 'local':
        return apply_cgroups(resources)
    
    # Dynamic scaling based on historical usage
    if task.get('auto_scale', False):
        historical = get_historical_usage(task['name'])
        resources = scale_resources(resources, historical)
    
    return resources
```

### Retry Policy with Exponential Backoff

```python
def should_retry(task, failure_reason, attempt):
    if attempt >= task.get('max_retries', 3):
        return False
    
    # Check if failure is retryable
    retryable_errors = [
        'ConnectionError',
        'TimeoutError',
        'HTTP 5xx',
        'K8s PodFailed',
        'EC2 Spot Instance Interruption'
    ]
    
    if failure_reason not in retryable_errors:
        return False
    
    # Calculate backoff: 2^attempt * initial_delay
    base_delay = task.get('retry_delay', 30)  # seconds
    max_delay = task.get('max_retry_delay', 600)  # 10 minutes
    delay = min(base_delay * (2 ** (attempt - 1)), max_delay)
    
    # Add jitter (±25%)
    jitter = delay * 0.25 * random.uniform(-1, 1)
    final_delay = delay + jitter
    
    time.sleep(final_delay)
    return True
```

## Golden Rules

1. **Pipeline Definitions Must Be Immutable**: Once a pipeline executes, its definition is stored with the execution. For reproducibility, never modify definitions of completed executions. Create new versions instead.

2. **External Dependencies Must Have Timeouts**: All external calls (APIs, databases) must specify explicit timeouts. Relying on OS defaults leads to hung tasks that block entire pipelines.

3. **Artifacts Must Be Immutable and Versioned**: Once an artifact is created, never modify it. Store in write-once storage (S3 with object lock, GCS with retention policy). Include execution ID in artifact paths.

4. **No Secrets in Pipeline Definitions**: Secrets must come from environment variables, Kubernetes secrets, or HashiCorp Vault. Never hardcode credentials. Use `{{ env.VAR_NAME }}` substitution.

5. **Each Task Must Be Atomic and Self-Contained**: Tasks should do one thing well and not depend on local state from previous runs. All inputs come from artifacts or variables; all outputs are artifacts.

6. **Failure Is Expected, Not Exceptional**: Design pipelines assuming tasks will fail. Include validation, retries, quarantines, and compensation transactions. A pipeline without error handling is broken by design.

7. **Resource Requests Must Be Realistic**: Over-allocating wastes money; under-allocating causes OOM kills. Use historical metrics to set CPU/memory requests. Always set memory limits.

8. **State Must Be Redis-Backed, Not Local**: Never store execution state on local filesystem. All state (including task progress) must be in Redis for crash recovery and multi-node coordination.

9. **Schedules Must Be Timezone-Aware**: Cron expressions without explicit timezone default to UTC. Always specify timezone to avoid DST surprises.

10. **Monitoring Is Not Optional**: Every pipeline must have:
    - Resource metrics (CPU, memory, I/O)
    - Duration metrics per task/stage
    - Failure rate tracking
    - Alerting on failures and SLA breaches
    - Integration with centralized logging (Loki/ELK)

11. **Rollback Must Be Fast and Tested**: For production pipelines, rollback procedures must be documented and tested quarterly. Rollback should complete in <50% of pipeline execution time.

12. **Parallelism Requires Clear Dependencies**: Use DAGs, not free-for-all parallel execution. Tasks that run in parallel must be truly independent; shared state without locking causes race conditions.

13. **Variables Must Be Validated**: All user-supplied variables (from `--vars` or `--env`) must have schema validation. Reject pipeline start if required variables missing or invalid types.

14. **Logs Must Be Structured and Centralized**: Use JSON logging. All logs must flow to centralized store with execution ID for traceability. Never rely on local log files.

15. **Changes Through PRs Only**: Pipeline definitions are code. All changes require pull request with review. No direct edits to production pipelines.

## Examples

### Example 1: Daily Batch ETL

**Command:**
```bash
orchestrate pipeline execute \
  --pipeline ./pipelines/sales_etl_daily.yaml \
  --env EXTRACTION_DATE=$(date -d 'yesterday' +%Y-%m-%d) \
  --vars ./config/feature_flags.json \
  --timeout 4h \
  --priority high \
  --trace-id $(uuidgen) \
  --monitor
```

**Pipeline Definition (`sales_etl_daily.yaml`):**
```yaml
version: "2.1"
name: "sales_etl_daily"
variables:
  EXTRACTION_DATE: "{{ ds }}"
  SOURCE_DB: "prod-sales-readonly"
  TARGET_DW: "analytics-prod"
  BATCH_SIZE: 10000
  
stages:
  - name: extract
    tasks:
      - name: extract_orders
        type: python_script
        script: ./src/extract_orders.py
        env:
          SOURCE_DB: "{{ SOURCE_DB }}"
          DATE: "{{ EXTRACTION_DATE }}"
          BATCH_SIZE: "{{ BATCH_SIZE }}"
        resources:
          cpu: 2.0
          memory: 4Gi
        timeout: 2h
        retry:
          max_attempts: 3
          backoff: exponential
        output_artifacts:
          - path: /tmp/orders_{{ EXTRACTION_DATE }}.jsonl
            type: dataset
            upload_to: s3://raw-sales-data/orders/date={{ EXTRACTION_DATE }}/
          
      - name: extract_customers
        type: python_script
        script: ./src/extract_customers.py
        dependencies: []
        env:
          SOURCE_DB: "{{ SOURCE_DB }}"
        resources:
          cpu: 1.0
          memory: 2Gi
        timeout: 1h
        output_artifacts:
          - path: /tmp/customers_{{ EXTRACTION_DATE }}.jsonl
            type: reference
            upload_to: s3://raw-sales-data/customers/date={{ EXTRACTION_DATE }}/
  
  - name: transform
    tasks:
      - name: transform_orders
        type: spark_job
        application: ./spark/transform_orders.py
        dependencies:
          - extract.extract_orders
          - extract.extract_customers
        resources:
          cpu: 8.0
          memory: 16Gi
          executor_instances: 4
        conf:
          spark.sql.adaptive.enabled: true
          spark.sql.adaptive.coalescePartitions.enabled: true
        input_artifacts:
          - s3://raw-sales-data/orders/date={{ EXTRACTION_DATE }}/*.jsonl
          - s3://raw-sales-data/customers/date={{ EXTRACTION_DATE }}/*.jsonl
        output_artifacts:
          - path: /tmp/transformed_orders_{{ EXTRACTION_DATE }}.parquet
            type: dataset
            upload_to: s3://transformed-sales-data/orders/date={{ EXTRACTION_DATE }}/
  
  - name: load
    tasks:
      - name: load_to_dw
        type: sql_script
        script: ./sql/load_orders.sql
        dependencies:
          - transform.transform_orders
        env:
          DW_CONNECTION: "postgresql://{{ env.DW_USER }}:{{ env.DW_PASSWORD }}@{{ TARGET_DW }}:5432/analytics"
          SOURCE_PATH: "s3://transformed-sales-data/orders/date={{ EXTRACTION_DATE }}/"
        resources:
          cpu: 2.0
          memory: 4Gi
        timeout: 1h
        post_hooks:
          - type: sql
            script: ./sql/refresh_materialized_views.sql
```

**Expected Output:**
```
[INFO] 2024-01-15T10:00:00Z - Starting orchestration execution: abc123-def456
[INFO] Pipeline: sales_etl_daily, Triggered by: user@example.com
[INFO] Validating pipeline definition...
[PASS] Validation succeeded (3 stages, 5 tasks)
[INFO] Allocating resources: extract (2 CPUs, 4Gi RAM each task)
[RUN] Stage 1/3: extract (parallel tasks: 2)
[RUN]   Task extract_orders (attempt 1/3) started at 10:00:05
[INFO]   Extracting orders for 2024-01-14, batch size: 10000
[RUN]   Task extract_customers (attempt 1/1) started at 10:00:06
[INFO]   Extracting customers (full refresh)
[DONE] extract_orders succeeded (1h 12m 34s) - 2.3M orders extracted
[DONE] extract_customers succeeded (42m 18s) - 458K customers extracted
[RUN] Stage 2/3: transform
[RUN]   Task transform_orders (attempt 1/1) started at 11:45:00
[INFO]   Spark job started with 4 executors
[DONE] transform_orders succeeded (35m 12s) - Parquet written to s3://...
[RUN] Stage 3/3: load
[RUN]   Task load_to_dw (attempt 1/3) started at 12:20:15
[DONE] load_to_dw succeeded (18m 45s) - 2.3M rows loaded
[INFO] Pipeline sales_etl_daily succeeded in 3h 6m 44s
[NOTIFY] Sending success notifications to: data-team@example.com, #data-pipelines
[ARCHIVE] Execution state saved to redis with 90d TTL
[ARCHIVE] Logs uploaded to: s3://pipeline-logs/abc123/combined.log.gz
[SUCCESS] Execution abc123-def456 completed successfully
```

### Example 2: ML Training Pipeline with Auto-Scaling

**Command:**
```bash
orchestrate pipeline execute \
  --pipeline ./ml/training_pipeline.yaml \
  --env MODEL_TYPE=resnet50 \
  --env TRAIN_DATA_PATH=s3://ml-data/training/2024-Q1/ \
  --env VAL_DATA_PATH=s3://ml-data/validation/2024-Q1/ \
  --vars ./config/hyperparameters.json \
  --resources '{"train_model": {"gpu": 4, "memory": "64Gi"}}' \
  --priority critical
```

**Pipeline Definition (`training_pipeline.yaml`):**
```yaml
version: "2.1"
name: "ml_training_resnet"
description: "Train ResNet50 model on image dataset"
variables:
  MODEL_TYPE: "{{ env.MODEL_TYPE }}"
  TRAIN_DATA_PATH: "{{ env.TRAIN_DATA_PATH }}"
  VAL_DATA_PATH: "{{ env.VAL_DATA_PATH }}"
  EPOCHS: 50
  BATCH_SIZE: 256
  LEARNING_RATE: 0.001
  
stages:
  - name: preprocessing
    tasks:
      - name: data_validation
        type: python_script
        script: ./ml/validate_data.py
        env:
          DATA_PATH: "{{ TRAIN_DATA_PATH }}"
          MIN_IMAGES: 100000
        resources:
          cpu: 4.0
          memory: 8Gi
        timeout: 2h
        retry: false
        validation_rules:
          - rule: image_count
            min: 100000
          - rule: class_distribution
            min_ratio: 0.01
            
      - name: feature_engineering
        type: spark_job
        application: ./spark/preprocess_images.py
        dependencies:
          - preprocessing.data_validation
        env:
          INPUT_PATH: "{{ TRAIN_DATA_PATH }}"
          OUTPUT_PATH: "s3://ml-preprocessed/train/$(date +%Y%m%d)/"
        resources:
          cpu: 16.0
          memory: 32Gi
          executor_instances: 8
        timeout: 6h
        
  - name: training
    tasks:
      - name: train_model
        type: kubernetes_job
        image: "registry.example.com/ml/trainer:{{ MODEL_TYPE }}-v2.1"
        dependencies:
          - preprocessing.feature_engineering
        env:
          TRAIN_DATA: "s3://ml-preprocessed/train/$(date +%Y%m%d)/"
          VAL_DATA: "{{ VAL_DATA_PATH }}"
          EPOCHS: "{{ EPOCHS }}"
          BATCH_SIZE: "{{ BATCH_SIZE }}"
          LEARNING_RATE: "{{ LEARNING_RATE }}"
          WANDB_API_KEY: "{{ secrets.WANDB_API_KEY }}"
          WANDB_PROJECT: "resnet-training"
        resources:
          cpu: 16.0  # Request for pod
          memory: 64Gi
          gpu: 4  # Must match node capacity
          limits:
            memory: 68Gi  # Allow 4Gi burst
        timeout: 48h
        retry:
          max_attempts: 2
        checkpoint:
          path: "s3://ml-checkpoints/{{ execution_id }}/"
          interval: "every 5 epochs"
        tensorboard:
          log_dir: "s3://ml-logs/tensorboard/{{ execution_id }}/"
        on_success:
          - type:Promote
            config:
              model_bucket: "s3://ml-models/production/"
              
      - name: evaluate_model
        type: python_script
        script: ./ml/evaluate.py
        dependencies:
          - training.train_model
        env:
          MODEL_PATH: "s3://ml-models/staging/{{ MODEL_TYPE }}/latest/"
          VAL_DATA_PATH: "{{ VAL_DATA_PATH }}"
          METRICS_OUTPUT: "./metrics.json"
        resources:
          cpu: 4.0
          memory: 16Gi
        timeout: 2h
        validation_rules:
          - rule: accuracy
            min: 0.85
            comparator: ">="
          - rule: inference_latency_p99
            max: 100  # ms
            comparator: "<="
        on_failure:
          - type:reject
            config:
              reason: "Model did not meet accuracy threshold"
              model_path: "s3://ml-models/staging/{{ MODEL_TYPE }}/latest/"
              
  - name: deployment
    tasks:
      - name: canary_deploy
        type: kubernetes_manifest
        manifest: ./k8s/canary-deployment.yaml
        dependencies:
          - training.evaluate_model
        template_vars:
          MODEL_VERSION: "{{ execution_id }}"
          IMAGE_TAG: "{{ MODEL_TYPE }}-{{ execution_id }}"
        resources:
          cpu: 1.0
          memory: 512Mi
        health_check:
          endpoint: "/health"
          timeout: 30s
          min_healthy: 90  # Percentage
        canary:
          initial_traffic: 10
          ramp_up: "every 5m, +10"
        rollback_on_failure: true
        timeout: 1h
```

**Output Sample:**
```
[INFO] Execution: abc789-xyz123 (ML Training ResNet50)
[VALIDATE] Pipeline structure valid
[ALLOCATE] GPU resources: requesting 4x NVIDIA A100 on GPU-enabled nodes
[RUN] Stage 1/3: preprocessing
[DONE] data_validation: 1.2M images validated (45m 12s)
[RUN]   feature_engineering: Spark job starting on 8 executors (32 cores, 256Gi RAM total)
[PROGRESS] Spark UI: http://spark-master:4040 (stages: 15/23 complete)
[DONE] feature_engineering: Preprocessing complete (4h 22m) - 1.2M images written
[RUN] Stage 2/3: training
[RUN]   train_model: Creating K8s job in namespace "ml-training"
[INFO]   Pod ml-abc789-xyz123-0 created on node gpu-pool-5
[STREAM] Training logs:
  Epoch 1/50, Loss: 4.256, Acc: 0.123
  Epoch 5/50, Loss: 2.891, Acc: 0.456, Checkpoint saved
  ...
  Epoch 50/50, Loss: 0.234, Acc: 0.923
[CHECKPOINT] Saved to s3://ml-checkpoints/abc789-xyz123/epoch_50.pt
[DONE] train_model: Training complete (47h 15m)
[RUN]   evaluate_model: Running inference on validation set (50K images)
[DONE] evaluate_model: Accuracy: 0.923, Latency p99: 87ms (1h 33m)
[PASS] Validation rules passed (accuracy > 0.85)
[RUN] Stage 3/3: deployment
[RUN]   canary_deploy: Creating K8s canary deployment
[PROGRESS] Rolling out canary: 10% → 20% → 30% (monitoring error rate)
[INFO] Error rate stable at 0.02%, continuing ramp-up
[DONE] canary_deploy: Canary at 100%, traffic shifted (45m)
[SUCCESS] Pipeline succeeded in 54h 10m
[NOTIFY] Slack: ":white_check_mark: ResNet50 model trained and deployed (acc: 92.3%)"
[ARCHIVE] Model artifacts: s3://ml-models/production/resnet50/abc789-xyz123/
[PROMOTE] Latest model pointer updated
```

### Example 3: Schedule Management

**Add a Schedule:**
```bash
orchestrate schedule add \
  --pipeline ./pipelines/customer_sync.yaml \
  --cron "0 */6 * * *" \
  --name "Customer Sync Every 6 Hours" \
  --max-retries 2 \
  --concurrency 1 \
  --timezone America/New_York \
  --skip-if-running \
  --notify-on-failure https://hooks.slack.com/services/...
```

**List Schedules:**
```bash
$ orchestrate schedule list --active-only --format table --next 5
NAME                        PIPELINE               SCHEDULE         NEXT RUN                 STATUS
Customer Sync Every 6h     customer_sync          0 */6 * * *      2024-01-16 00:00 EST    Active
Daily ETL 2AM UTC          daily_sales_etl        0 2 * * *        2024-01-16 02:00 UTC    Active
Hourly Metrics             hourly_metrics        0 * * * *        2024-01-15 13:00 UTC    Active
```

**Manually Trigger Scheduled Run (with different variables):**
```bash
orchestrate pipeline execute \
  --pipeline ./pipelines/customer_sync.yaml \
  --env FORCE_FULL_SYNC=true \
  --priority high \
  --trace-id manual-$(date +%s)
```

### Example 4: State Recovery and Rollback

**Scenario**: A pipeline failure due to transient error; after fix, resume from last successful stage.

**Check State:**
```bash
$ orchestrate state get --execution-id abc123-def456 --format json --compact
{
  "execution_id": "abc123-def456",
  "status": "failed",
  "pipeline": "daily_sales_etl",
  "started_at": "2024-01-15T02:00:00Z",
  "completed_at": "2024-01-15T04:30:00Z",
  "stages": [
    {"name": "extract", "status": "succeeded", "tasks": [...]},
    {"name": "transform", "status": "succeeded", "tasks": [...]},
    {"name": "validate", "status": "failed", 
     "tasks": [{"id": "data_quality", "status": "failed", "error": "API timeout"}]}
  ]
}
```

**Rollback to Checkpoint (Before Failure Stage):**
```bash
# The pipeline completed, but we want to restart from validate stage
orchestrate rollback execute \
  --execution-id abc123-def456 \
  --to-stage validate \
  --strategy restart \
  --include-artifacts \
  --verify \
  --notify https://alerts.example.com/rollback_complete
```

**Or rollback to previous successful execution:**
```bash
# Find previous successful execution ID
$ orchestrate history list --pipeline daily_sales_etl --status succeeded --limit 5 | head -1
xyz789-uvw456  2024-01-14 02:00 UTC  succeeded  2h 45m

# Rollback state to match that execution
orchestrate rollback execute \
  --execution-id abc123-def456 \
  --to-execution-id xyz789-uvw456 \
  --strategy checkpoint \
  --force \
  --dry-run
```

**Dry-run output:**
```
[Dry Run] Rollback Plan for execution abc123-def456:
1. Restore Redis state from execution xyz789-uvw456
2. Download artifacts:
   - s3://transformed-sales-data/orders/date=2024-01-13/xyz789_artifacts.tar.gz
   -> /tmp/orchestration/artifacts/
3. Reset stage status: load → PENDING
4. Keep execution ID: abc123-ded456 (new run)
5. Verification checks:
   - Validate artifact integrity (SHA256)
   - Confirm dependency stage outputs exist
6. Estimated rollback time: 15m
[DRY-RUN] No changes applied (remove --dry-run to execute)
```

## Dependencies and Requirements

### System Dependencies

- **Python 3.9+**: Core runtime
- **Redis 6.2+**: State persistence and pub/sub
- **Docker 20.10+**: Container execution (if using Docker executor)
- **Kubernetes 1.24+**: K8s job orchestration (if using K8s executor)
- **AWS CLI 2.x**: S3 artifact storage and Batch jobs (optional)
- **Git 2.30+**: Pipeline definition version control

### Python Packages

```bash
pip install \
  pyyaml==6.0 \
  croniter==1.4.7 \
  pydantic==2.4.2 \
  redis==4.6.0 \
  boto3==1.28.62 \
  kubernetes==26.1.0 \
  pyspark==3.5.0 \
  sqlalchemy==2.0.23 \
  requests==2.31.0 \
  click==8.1.7
```

### Configuration File (`~/.openclaw/config/auto-orchestrator.yaml`)

```yaml
orchestrator:
  redis_url: "redis://localhost:6379/0"
  workdir: "/var/lib/orchestration"
  log_level: "INFO"
  executor: "k8s"  # local, docker, k8s, aws-batch
  
  state:
    ttl_days: 90
    compression: true
    archive_after_days: 30
    
  resources:
    default_cpu: 1.0
    default_memory: "2Gi"
    max_concurrent_tasks: 100
    
  retry:
    default_max_attempts: 3
    default_backoff: "exponential"
    initial_delay_seconds: 30
    max_delay_seconds: 600
    
  notifications:
    smtp:
      host: "smtp.example.com"
      port: 587
      from: "orchestrator@example.com"
    slack_webhook: "https://hooks.slack.com/services/..."
    pagerduty_service_key: "{{ env.PAGERDUTY_KEY }}"
    
  storage:
    artifact_store: "s3"  # s3, gcs, azure, local
    s3_bucket: "orchestration-artifacts"
    s3_region: "us-east-1"
    local_path: "/data/orchestration/artifacts"
    
  monitoring:
    prometheus_endpoint: "http://prometheus:9090"
    grafana_dashboard: "orchestration-overview"
    alert_webhooks:
      - "https://alerts.example.com/pipelines"
      
  executor_config:
    docker:
      network_mode: "bridge"
      default_image: "python:3.9-slim"
      registry: "registry.example.com"
    k8s:
      namespace: "data-pipelines"
      service_account: "orchestrator"
      image_pull_secrets: ["registry-creds"]
      node_selectors:
        gpu: "nvidia.com/gpu.present"
      tolerations:
        - key: "dedicated"
          operator: "Equal"
          value: "orchestration"
          effect: "NoSchedule"
    aws_batch:
      job_queue: "orchestration-queue"
      job_definition: "orchestration-job:1"
      compute_environment: "orchestration-env"
```

### Security Requirements

1. **Principle of Least Privilege**:
   - Docker containers run as non-root users (UID 1000+)
   - K8s pods use dedicated service accounts with minimal RBAC
   - AWS IAM roles scoped to specific S3 buckets and DynamoDB tables

2. **Secrets Management**:
   - Never store secrets in pipeline definitions
   - Use Kubernetes secrets, AWS Secrets Manager, or HashiCorp Vault
   - Rotate secrets automatically (30-day rotation)
   - Audit all secret access

3. **Network Policies**:
   - K8s: Restrict pod-to-pod communication via NetworkPolicy
   - Docker: Use custom bridge networks with firewall rules
   - Egress only to whitelisted external endpoints

4. **Resource Quotas**:
   - Per-namespace CPU/memory limits in K8s
   - Docker daemon resource limits
   - AWS Batch compute environment limits

5. **Artifact Encryption**:
   - S3: SSE-S3 or SSE-KMS with rotation
   - Transit: TLS 1.3 for all network traffic
   - At-rest: Full-disk encryption for local storage

## Verification Steps

### After Pipeline Execution

1. **Check Execution State**:
```bash
# Verify execution record
orchestrate state get --execution-id abc123-def456 --format json | jq '.status'

# Check all stages completed
orchestrate state get --execution-id abc123-def456 --format json | jq '.stages[].status'
```

2. **Validate Artifact Integrity**:
```bash
# Compare checksums
aws s3api head-object \
  --bucket orchestration-artifacts \
  --key "executions/abc123-def456/artifacts/transformed_orders.parquet" \
  --query 'Metadata.checksum' | \
  sha256sum -c -

# Verify all expected artifacts exist
EXPECTED=$(yq '.stages[].tasks[].output_artifacts[].upload_to' pipeline.yaml | wc -l)
ACTUAL=$(aws s3 ls s3://transformed-sales-data/ | grep abc123-def456 | wc -l)
test $EXPECTED -eq $ACTUAL && echo "All artifacts uploaded" || echo "Artifact count mismatch"
```

3. **Confirm Data Quality**:
```bash
# If data quality task ran, check its metrics
orchestrate state get --execution-id abc123-def456 \
  --format json | jq '.stages[] | select(.name=="validate") | .tasks[] | select(.name=="data_quality_checks") | .outputs.metrics'
```

4. **Verify Notifications Sent**:
```bash
# Check notification log (if using Slack)
grep "abc123-def456" /var/log/orchestrator/notifications.log | tail -5
```

5. **Review Resource Usage**:
```bash
# Query Prometheus metrics
curl -s "http://prometheus:9090/api/v1/query?query=orchestrator_task_duration_seconds{execution_id=\"abc123-def456\"}" | jq '.data.result[] | {task: .metric.task, duration: .value[1]}'

# Check for over-provisioning
curl -s "http://prometheus:9090/api/v1/query?query=orchestrator_task_cpu_usage_percent{execution_id=\"abc123-def456\"}" | jq -r '.data.result[] | select(.value[1] | tonumber < 50) | "Underutilized: \(.metric.task) \(.value[1])%"'
```

### Pre-Execution Validation Checklist

```bash
# 1. Validate pipeline syntax
orchestrate validate pipeline --pipeline ./new_pipeline.yaml --strict
if [ $? -ne 0 ]; then exit 1; fi

# 2. Check all scripts exist
while read -r script; do
  test -f "$script" || { echo "Missing script: $script"; exit 1; }
done < <(yq '.stages[].tasks[].script' new_pipeline.yaml | grep -v null)

# 3. Verify external connections
orchestrate validate pipeline \
  --pipeline ./new_pipeline.yaml \
  --check-connections \
  --timeout 30s

# 4. Check resource quotas
orchestrate resource check \
  --pipeline ./new_pipeline.yaml \
  --executor k8s

# 5. Test variable substitution
export EXTRACTION_DATE="2024-01-15"
orchestrate validate pipeline \
  --pipeline ./new_pipeline.yaml \
  --env EXTRACTION_DATE \
  --format json | jq '.variables'
```

### Health Check for Orchestrator Service

```bash
#!/bin/bash
# Check orchestrator daemon status

# 1. Check Redis connectivity
redis-cli -h localhost -p 6379 ping || { echo "Redis down"; exit 1; }

# 2. Check orchestrator process
systemctl is-active --quiet orchestrator || { echo "Orchestrator service stopped"; exit 1; }

# 3. Check recent executions haven't all failed
FAILED_24H=$(orchestrate history list --last 24h --status failed | wc -l)
SUCCESS_24H=$(orchestrate history list --last 24h --status succeeded | wc -l)
RATIO=$(( FAILED_24H * 100 / (FAILED_24H + SUCCESS_24H) ))
if [ $RATIO -gt 20 ]; then
  echo "High failure rate: ${RATIO}% in last 24h"
  exit 2
fi

# 4. Check disk space for artifacts
DISK_USAGE=$(df --output=pcent /var/lib/orchestration | tail -1 | tr -d '%')
if [ $DISK_USAGE -gt 85 ]; then
  echo "Disk usage critical: ${DISK_USAGE}%"
  exit 3
fi

echo "Orchestrator healthy"
```

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: "Pipeline validation failed: stage tasks reference non-existent script"

**Cause**: Script path in pipeline definition doesn't exist or is relative to wrong directory.

**Fix**:
```bash
# Ensure you're in pipeline directory
cd /path/to/pipeline/directory

# Verify all script paths
yq '.stages[].tasks[].script' pipeline.yaml | while read script; do
  test -f "$script" || echo "Missing: $script"
done

# Use absolute paths or ensure pipeline is executed from correct workdir
orchestrate pipeline execute \
  --pipeline /absolute/path/to/pipeline.yaml \
  --workdir /absolute/path/to/
```

#### Issue 2: "Kubernetes pod failed: OOMKilled"

**Cause**: Task memory request too low; container exceeded limit.

**Fix**:
```bash
# Check actual memory usage from logs
kubectl logs pod/ml-abc123 --previous | grep " Memory "

# Adjust resources in pipeline
# Increase memory request AND limit (limit should be ~10-20% higher than request)
resources:
  memory: "16Gi"  # Request
  limits:
    memory: "18Gi"  # Allow burst
```

**Prevention**: Set historical monitoring to auto-scale memory:
```yaml
resources:
  memory: "auto"  # Uses 95th percentile from historical runs
  limits:
    memory: "auto+2Gi"
```

#### Issue 3: "Task stuck in RUNNING state for > timeout"

**Cause**: Task exceeded its timeout or hung waiting for external resource.

**Diagnosis**:
```bash
# View current task state
orchestrate state get --execution-id abc123 --format json | jq '.stages[] | select(.status=="running") | .tasks[] | select(.status=="running")'

# Check logs via Redis (or S3 if uploaded)
redis-cli HGET orchestration:execution:abc123 tasks:extract_orders:logs | head -50

# If K8s pod
kubectl describe pod/ml-abc123 --namespace data-pipelines
kubectl logs pod/ml-abc123 --previous  # Previous container if restarted
```

**Fix**:
```bash
# Increase timeout in pipeline
tasks:
  - name: long_running_task
    timeout: 6h  # Instead of 2h

# Or manually cancel if stuck
orchestrate state set \
  --execution-id abc123 \
  --task stuck_task_id \
  --status failed \
  --message "Manual cancel - exceeded SLA" \
  --force
```

#### Issue 4: "Redis connection refused"

**Cause**: Redis not running or wrong connection string.

**Fix**:
```bash
# Check Redis status
systemctl status redis
sudo redis-cli ping

# Fix Redis URL in config
orchestrate config set \
  --key redis_url \
  --value "redis://localhost:6379/0" \
  --type string

# If Redis needs authentication
orchestrate config set \
  --key redis_url \
  --value "rediss://:password@redis-host:6379/0" \
  --type string
```

#### Issue 5: "Pipeline failed: variable substitution error - undefined variable"

**Cause**: Required variable not provided via `--env` or `--vars`.

**Fix**:
```bash
# Check required variables
REQUIRED=$(yq '.variables | keys' pipeline.yaml)
PROVIDED=$(env | grep -E '^(EXTRACTION_DATE|...)=' | cut -d= -f1)

echo "Missing: $(comm -23 <(echo "$REQUIRED" | sort) <(echo "$PROVIDED" | sort))"

# Provide via vars file
cat > overrides.json << 'EOF'
{
  "EXTRACTION_DATE": "2024-01-15",
  "SALES_API_ENDPOINT": "https://api.sales.com/v2"
}
EOF

orchestrate pipeline execute \
  --pipeline pipeline.yaml \
  --vars overrides.json
```

#### Issue 6: "Task failed: permission denied on S3 bucket"

**Cause**: IAM role or credentials don't have access to S3 bucket.

**Fix**:
1. Check IAM policy attached to role:
```bash
aws iam get-role --role-name orchestration-task-role | jq '.Role.AssumeRolePolicyDocument'
aws iam list-attached-role-policies --role-name orchestration-task-role
```

2. Verify bucket policy allows access:
```bash
aws s3api get-bucket-policy --bucket raw-sales-data | jq .
```

3. Test with AWS CLI (using same credentials):
```bash
aws s3 ls s3://raw-sales-data/ --recursive --max-items 1
```

4. Update pipeline to use correct credentials:
```yaml
tasks:
  - name: extract_orders
    env:
      AWS_ACCESS_KEY_ID: "{{ secrets.AWS_ACCESS_KEY }}"
      AWS_SECRET_ACCESS_KEY: "{{ secrets.AWS_SECRET_KEY }}"
      AWS_DEFAULT_REGION: "us-east-1"
    # OR use IAM role if running on EC2/EKS
```

#### Issue 7: "Stage failed: task retry limit exceeded"

**Cause**: Retry policy exhausted without success.

**Diagnosis**:
```bash
# Check error pattern
orchestrate state get --execution-id abc123 --format json | jq '.stages[].tasks[] | select(.status=="failed") | {id: .id, error: .error, attempt: .attempt}'

# Look at logs to determine if transient or permanent
orchestrate logs --execution-id abc123 --task stuck_task --lines 50
```

**Fix**:
- **Transient errors** (network, API rate limits): Increase retry count or backoff.
- **Permanent errors** (bad data, invalid query): Fix pipeline code and re-run.

To skip failed task and continue (if safe):
```bash
orchestrate pipeline execute \
  --pipeline pipeline.yaml \
  --skip-tasks stuck_task_id,another_failed_task \
  --force
```

#### Issue 8: "K8s pod pending indefinitely"

**Cause**: Insufficient cluster resources or node selector mismatch.

**Diagnosis**:
```bash
kubectl describe pod/ml-abc123 --namespace data-pipelines | grep -A 10 "Events:"

kubectl get nodes -o json | jq '.items[] | {name: .metadata.name, allocatable: .status.allocatable}'
```

**Fix**:
```yaml
# Adjust resource requests to match cluster capacity
resources:
  cpu: "2.0"  # Not "16.0" if cluster has limited cores
  memory: "4Gi"
  
  # Or remove restrictive node selectors
  # node_selector: {}  # Allow any node
```

**Temporary workaround**: Increase cluster size or use spot instances.

#### Issue 9: "Artifact upload failed: AccessDenied"

**Cause**: Orchestrator service account lacks write permissions to storage bucket.

**Fix**:
```bash
# Check current IAM role
aws sts get-caller-identity

# Update bucket policy to allow orchestration role
cat > bucket-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/orchestration-task-role"
      },
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::orchestration-artifacts/*"
    }
  ]
}
EOF

aws s3api put-bucket-policy --bucket orchestration-artifacts --policy file://bucket-policy.json
```

#### Issue 10: "Pipeline succeeded but data looks stale/incomplete"

**Cause**: Logic error in extraction/transformation code, not caught by validation.

**Debugging**:
```bash
# Compare row counts with previous run
PREV_COUNT=$(orchestrate state get --execution-id previous_run --format json | jq -r '.stages[].tasks[] | select(.name=="load_to_dw") | .outputs.row_count')
CUR_COUNT=$(orchestrate state get --execution-id abc123 --format json | jq -r '.stages[].tasks[] | select(.name=="load_to_dw") | .outputs.row_count')

echo "Previous: $PREV_COUNT, Current: $CUR_COUNT"

# If using data quality validation, check its metrics
orchestrate state get --execution-id abc123 --format json | jq '.stages[] | select(.name=="validate") | .tasks[] | .validation_results'

# Run data diff manually (if Delta Lake/Hudi)
spark-submit --class DataDiff ./tools/data-diff.jar \
  --base s3://dw/table/date=2024-01-13/ \
  --target s3://dw/table/date=2024-01-14/ \
  --output diff_report.html
```

### Performance Tuning

1. **Slow Task**:
   - Profile code: add timing logs or use `cProfile`
   - Increase allocated CPU/memory
   - Parallelize within task (multi-threading, Spark partitions)
   - Consider splitting into multiple pipeline tasks

2. **Resource Waste**:
   - Enable auto-scaling based on historical metrics:
   ```yaml
   resources:
     cpu: "auto"  # Uses 80th percentile from last 10 runs
     memory: "auto*1.2"  # 20% buffer
   ```

3. **Long Pipeline Duration**:
   - Increase `max_concurrent` in pipeline config
   - Identify and eliminate sequential dependencies
   - Use `parallel: true` at stage level

### Log Debugging Commands

```bash
# Stream live logs (if running)
orchestrate logs --execution-id abc123 --follow

# Get logs from completed task
orchestrate logs --execution-id abc123 --task extract_orders --full > extract_orders.log

# Search logs for error pattern
orchestrate logs --execution-id abc123 --task extract_orders | grep -i "timeout\|error\|exception" | head -20

# Unified log tail with timestamps
orchestrate logs --execution-id abc123 --timestamps --format json | jq -r '"\(.timestamp) \(.task) \(.level): \(.message)"'
```

## Rollback Commands

### Point-in-Time Rollback

Restore pipeline state to a previous successful execution or checkpoint:

```bash
# Rollback to previous successful execution (full state restore)
orchestrate rollback execute \
  --execution-id abc123-def456 \
  --to-execution-id xyz789-uvw456 \
  --strategy checkpoint \
  --include-artifacts \
  --verify \
  --force

# Rollback to specific timestamp (e.g., before bad data loaded)
orchestrate rollback execute \
  --execution-id abc123-def456 \
  --to-timestamp "2024-01-15T14:30:00Z" \
  --strategy revert \
  --dry-run

# Rollback specific stage only (partial rollback)
orchestrate rollback execute \
  --execution-id abc123-def456 \
  --to-execution-id xyz789-uvw456 \
  --stage validate \
  --strategy restart \
  --force
```

### Artifact Recovery

Manually restore artifacts from archive without affecting execution state:

```bash
# Download artifact from specific execution
orchestrate artifact download \
  --execution-id xyz789-uvw456 \
  --artifact transformed_orders.parquet \
  --output ./recovered/transformed_orders.parquet

# Or download entire artifact set
orchestrate artifact download-all \
  --execution-id xyz789-uvw456 \
  --output ./recovered/
```

### Database Rollback (if using SQL scripts with transactions)

```bash
# Orchestrator can automatically wrap SQL tasks in transactions if configured
# Manual database rollback (PostgreSQL example):
orchestrate db rollback \
  --target-dw analytics-prod \
  --to-timestamp "2024-01-15 04:30:00 UTC" \
  --tables orders,customers,order_items \
  --dry-run

# Output:
# [DRY-RUN] Would execute:
# BEGIN;
# DELETE FROM orders WHERE created_at >= '2024-01-15 04:30:00';
# DELETE FROM order_items WHERE order_id IN (SELECT id FROM orders WHERE ...);
# DELETE FROM customers WHERE updated_at >= '2024-01-15 04:30:00';
# COMMIT;
# [DRY-RUN] Rows to delete: orders=2.3M, order_items=5.1M, customers=458K
```

### State Cleanup After Failed Rollback

If rollback leaves inconsistent state:

```bash
# Force cleanup partial execution
orchestrate state cleanup \
  --execution-id abc123-def456 \
  --force

# Reset specific task to retry
orchestrate state set \
  --execution-id abc123-def456 \
  --task load_to_dw \
  --status pending \
  --message "Retry after manual fix" \
  --force
```

### Cluster/Infrastructure Rollback

If pipeline deployed infrastructure that needs teardown:

```bash
# Tear down K8s resources created by pipeline
orchestrate infra destroy \
  --execution-id abc123-def456 \
  --resources deployment,service,configmap \
  --namespace data-pipelines \
  --dry-run

# Rollback Terraform state (if pipeline applied Terraform)
orchestrate terraform rollback \
  --execution-id abc123-def456 \
  --state-file ./terraform.tfstate \
  --backup-file ./terraform.tfstate.backup \
  --plan ./rollback_plan
```

### Full System Rollback (Emergency)

In case of catastrophic failure affecting multiple executions:

```bash
# Restore Redis from backup (last 24h)
redis-cli -h backup-redis -p 6379 --rdb /tmp/restore.rdb
systemctl restart redis

# Replay execution from archival state
orchestrate state restore-archive \
  --archive s3://orchestration-archives/2024-01-15/abc123.parquet \
  --dry-run
```

### Rollback Verification Checklist

After any rollback, verify:

```bash
# 1. Execution state is consistent
orchestrate state get --execution-id abc123 --format json | jq '.status'  # Should be succeeded or running

# 2. All expected artifacts exist
EXPECTED_ARTIFACTS=$(yq '.stages[].tasks[].output_artifacts[].upload_to' pipeline.yaml | wc -l)
ACTUAL_ARTIFACTS=$(aws s3 ls s3://transformed-sales-data/ | grep abc123 | wc -l)
test $EXPECTED_ARTIFACTS -eq $ACTUAL_ARTIFACTS || echo "Artifact count mismatch!"

# 3. Downstream systems received data
curl -s https://dw-healthcheck.example.com/status | jq '.last_updated > "2024-01-15T04:00:00Z"'

# 4. No orphaned resources
kubectl get pods --namespace data-pipelines --field-selector status.phase=Running | grep abc123 || echo "No orphaned pods"

# 5. Notifications sent
grep "Rollback completed" /var/log/orchestrator/notifications.log | tail -1
```

### Rollback Strategy Comparison

| Strategy | Use Case | Duration | Risk |
|----------|----------|----------|------|
| `checkpoint` | Hardware failure mid-pipeline | Low (15m) | Minimal - restores exact state |
| `restart` | Logical error in last stage | Medium (full stage runtime) | Medium - re-runs from stage |
| `revert` | Bad data landed in warehouse | High (full pipeline) | High - may need manual cleanup |

**Recommendation**: Prefer `checkpoint` when available. Use `revert` only as last resort after understanding data contamination scope.

### Rollback Dry-Run Preview

Always preview rollback impacts:

```bash
orchestrate rollback execute \
  --execution-id abc123 \
  --to-execution-id xyz789 \
  --dry-run \
  --verbose

# Typical output:
[DRY-RUN] Rollback plan:
1. Restore Redis keys: 147 keys from execution xyz789
2. Delete Redis keys for abc123: 89 keys
3. Download artifacts (2.3GB):
   - s3://.../transformed_orders.parquet (1.8GB)
   - s3://.../data_quality_report.json (15MB)
4. Delete S3 artifacts created by abc123: 47 objects
5. Delete K8s pods: ml-abc123-*
6. NOT destroying: persistent volumes, RDS instances
7. Verification steps will run after restore
[DRY-RUN] Estimated time: 12m
[DRY-RUN] Risk level: LOW (no schema changes detected)
```

---

**Important**: Always test rollback procedures in non-production environments quarterly. Document pipeline-specific rollback steps in the pipeline's README.
```
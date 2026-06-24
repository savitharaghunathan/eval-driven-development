# Data Workflow Archetype

## Signature Patterns

ETL, data transformations, scheduling, data quality checks, reporting, analytics.

**Examples:** Data pipeline, report generator, analytics automation, data migration.

## Extraction Checklist

When analyzing a spec for data workflow systems, look for:

- Data sources and sinks (where does data come from and go?)
- Transformation logic (what computations or restructuring?)
- Scheduling and orchestration (what triggers runs?)
- Data quality contracts (what guarantees does the pipeline make?)
- Error handling (what happens on partial failure, bad input?)
- Idempotency requirements (running twice = same result?)
- Lineage/provenance requirements (can you trace where data came from?)

## Default Eval Dimensions

- Transformation correctness (output matches expected)
- Idempotency (running twice produces same result)
- Schema compliance (output conforms to expected schema)
- Error handling coverage (graceful failure on malformed input)
- Throughput (processes within SLA)
- Data freshness (output available within required window)
- Lineage accuracy (provenance tracked correctly)
- Edge case handling (nulls, empty sets, large inputs, encoding issues)

## Default Grader Patterns

| Requirement Type | Grader Type | Notes |
| ---------------- | ----------- | ----- |
| Transformation correctness | code-based | Compare output to expected |
| Idempotency | outcome verification | Run twice, compare state |
| Schema compliance | code-based | JSON schema / type validation |
| Error handling coverage | outcome verification | Inject bad input, check graceful failure |
| Throughput | code-based | Measure time |
| Data freshness | code-based | Check timestamps |
| Lineage accuracy | code-based | Verify provenance records |
| Edge case handling | code-based | Test with nulls, empties, encoding issues |

## Common Failure Modes

- Pipeline silently drops records on malformed input instead of reporting errors
- Running the pipeline twice creates duplicate records (non-idempotent)
- Schema changes upstream break the pipeline without clear error messages
- Lineage/provenance tracking misses intermediate transformations
- Pipeline succeeds on small test data but fails or hangs on production-scale data
- Null handling is inconsistent across transformations

## Example Eval Tasks

```yaml
- id: idempotency-check
  requirement: Running the pipeline twice produces the same result
  description: Execute the full pipeline, then execute it again without clearing state
  input: Standard input dataset of 100 records
  expected_behavior: Output after second run is identical to output after first run
  reference_solution: Both runs produce exactly 100 output records with identical content and no duplicates
  grader_type: outcome
  grader_logic: Run pipeline twice. Diff output state after run 1 vs run 2. PASS if identical, FAIL if any differences.
  category: capability
  priority: P0

- id: graceful-failure-on-malformed-input
  requirement: Pipeline handles malformed records without crashing
  description: Include 5 malformed records (missing required fields, wrong types) in a batch of 100
  input: 95 valid records + 5 records with various schema violations
  expected_behavior: Pipeline processes 95 valid records, logs errors for 5 invalid records, does not crash
  reference_solution: Output contains 95 records. Error log contains 5 entries with record IDs and error descriptions.
  grader_type: outcome
  grader_logic: Check output record count = 95. Check error log contains 5 entries. Check pipeline exit code is success (not crash).
  category: capability
  priority: P1
```

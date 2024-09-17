# gpt-batch-manager

The [OpenAI Batch API] is great for submitting batch jobs, but it has some strict [limits]:

- gpt-4o: 90,000 tokens (Tier 1), 1.35M (Tier 2)
- gpt-4o-mini: 2,000,000 tokens (Tier 1), 20M (Tier 2)

If you want to submit larger batches than your current tier allows, you'll have to split
the work into multiple batches. This project helps you do that by providing two tools:

1. `gpt-batch-splitter`: Take one or more JSONL files and split them into shards that
   fit within your tier batch queue limit.
2. `gpt-batch-manager`: Submit multiple batches to OpenAI, one at a time.

## Usage

Install:

    pip install gpt-batch-manager

To use the batch manager, you'll need to set an `OPENAI_API_KEY` environment variable.
The best way to do this is by creating a `.env` file in your working directory. See the
[OpenAI Python API docs][dotenv] for details.

### Batch Splitter

Say your tasks are in `largefile1.jsonl` and `largefile2.jsonl`. You want to submit them
to the OpenAI batch API for gpt-4o-mini and you're on Tier 1, so you need to make them
fit within a 2M token limit.

To produce shards with fewer than 2M tokens, run:

    gpt-batch-splitter 1900000 largefile1.jsonl largefile2.jsonl

This will [estimate] the number of tokens in each request and divvy them up accordingly.
It will produce output files like:

    shard-000-of-079.jsonl
    shard-001-of-079.jsonl
    ...
    shard-078-of-079.jsonl

Since the splitter has to estimate the number of tokens in each request, it's best to
use a number somewhat below the limit.

### Batch Manager

To stay within your batch limit, you have to submit batch files one-by-one. This is what
`gpt-batch-manager` does. Pass it a set of JSONL files:

    gpt-batch-manager shard-???-of-???.jsonl

Make sure you have an `OPENAI_API_KEY` environment variable set (see above). This will
report the status of each batch as it progresses. Output will eventually appear in:

    shard-000-of-079.output.jsonl
    shard-001-of-079.output.jsonl
    ...

This process is fully resumable. You can Ctrl-C it at any time and it will pick up where
it left off. It stores state in `/tmp/batch-status.json`. If something goes wrong, you
could try deleting that file to reset.

[OpenAI Batch API]: https://platform.openai.com/docs/guides/batch
[limits]: https://platform.openai.com/docs/guides/rate-limits/usage-tiers?context=tier-one
[dotenv]: https://github.com/openai/openai-python?tab=readme-ov-file#usage
[estimate]: https://openai.com/api/pricing/

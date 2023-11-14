---

title: Why you should specify a production environment in dbt Cloud
description: "The bottom line: You should split your Environments in dbt Cloud based on their purposes (e.g. Production and Staging/CI) and mark one environment as Production. This will improve your CI experience and enable you to use dbt Explorer."
slug: specify-prod-environment

authors: [joel_labes]

tags: [dbt Cloud]
hide_table_of_contents: false

date: 2023-11-14
is_featured: false

---

:::tip The Bottom Line:
You should [split your Jobs](#how) across Environments in dbt Cloud based on their purposes (e.g. Production and Staging/CI) and set one environment as Production. This will improve your CI experience and enable you to use dbt Explorer.
:::

[Environmental segmentation](/docs/environments-in-dbt) has always been an important part of the analytics engineering workflow:

- When developing new models you can [process a smaller subset of your data](/reference/dbt-jinja-functions/target#use-targetname-to-limit-data-in-dev) by using `target.name` or an environment variable.
- By building your production-grade models into [a different schema and database](https://docs.getdbt.com/docs/build/custom-schemas#managing-environments), you can experiment in peace without being worried that your changes will accidentally impact downstream users.
- Using dedicated credentials for production runs, instead of an analytics engineer's individual dev credentials, ensures that things don't break when that long-tenured employee finally hangs up their IDE.

Historically, dbt Cloud required a separate environment for _Development_, but was otherwise unopinionated in how you configured your account. This mostly just worked – as long as you didn't have anything more complex than a CI job mixed in with a couple of production jobs – because important constructs like deferral in CI and documentation were only ever tied to a single job.

But as companies' dbt deployments have grown more complex, it doesn't make sense to assume that a single job is enough anymore. We need to exchange a job-oriented strategy for a more mature and scalable environment-centric view of the world. To support this, a recent change in dbt Cloud enables project administrators to [mark one of their environments as the Production environment](/docs/deploy/deploy-environments#set-as-production-environment-beta), just as has long been possible for the Development environment.

Explicitly separating your Production workloads lets dbt Cloud be smarter with the metadata it creates, and is particularly important for two new features: dbt Explorer and the revised CI workflows.

<!-- truncate -->

## Make sure dbt Explorer always has the freshest information available

**The old way**: Your dbt docs site was based on a single job's run.

**The new way**: dbt Explorer uses metadata from across every invocation in a defined Production environment to build the richest and most up-to-date understanding of your project.

Because dbt docs could only be updated by a single predetermined job, users who needed their documentation to immediately reflect changes deployed throughout the day (regardless of which job executed them) would find themselves forced to run a dedicated job which did nothing other than run `dbt docs generate` on a regular schedule.

The Discovery API that powers dbt Explorer ingests all metadata generated by any dbt invocation, which means that it can always be up to date with the applied state of your project. However it doesn't make sense for dbt Explorer to show docs based on a PR that hasn't been merged yet.

To avoid this conflation, you need to mark an environment as the Production environment. All runs completed in _that_ environment will contribute to dbt Explorer's, while others will be excluded. (Future versions of Explorer will support environment selection, so that you can preview your documentation changes as well.)

## Run Slimmer CI than ever with environment-level deferral

**The old way**: [Slim CI](/guides/set-up-ci?step=2) deferred to a single job, and would only detect changes as of that job's last build time.

**The new way**: Changes are detected regardless of the job they were deployed in, removing false positives and overbuilding of models in CI.

Just like dbt docs, relying on a single job to define your state for comparison purposes leads to a choice between unnecessarily rebuilding models which were deployed by another job, or creating a dedicated job that runs `dbt compile` on repeat to keep on top of all changes.

By using the environment as the arbiter of state, any time a change is made to your Production deployment it will immediately be taken into consideration by subsequent Slim CI runs.

## The easiest way to break apart your jobs {#how}

<Lightbox src="/img/blog/2023-11-06-differentiate-prod-and-staging-environments/data-landscape.png" alt="A chart showing the interplay of Data Warehouse, git repo and dbt Cloud project across Dev, CI and Prod environments." title="Your organization's data landscape should separate Dev, CI and Prod environments. To achieve this, configure your data warehouse, git repo and dbt Cloud account as shown above." width="100%"/>

For most projects, changing from a job-centric to environment-centric approach to metadata is straightforward and immediately pays dividends as described above. Assuming that your Staging/CI and Production jobs are currently intermingled, you can extricate them as follows:

1. Create a new dbt Cloud environment called Staging
2. For each job that belongs to the Staging environment, edit the job and update its environment
3. Tick the ["Mark as Production environment" box](/docs/deploy/deploy-environments#set-as-production-environment-beta) in your original environment's settings

## Conclusion

Until very recently, I only thought of Environments in dbt Cloud as a way to use different authentication credentials in different contexts. And until very recently, I was mostly right.

Not anymore. The metadata dbt creates is critical for effective data teams – whether you're concerned about cost savings, discoverability, increased development speed or reliable results across your organization – but is only fully effective if it's segmented by the environment that created it.

Take a few minutes to clean up your environments - it'll make all the difference.
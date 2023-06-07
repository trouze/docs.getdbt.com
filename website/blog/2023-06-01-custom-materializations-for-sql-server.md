---
title: "Bring your custom SSIS transformations to dbt via Custom Materializations"
description: "For folks coming from a SQL Server background, Analytics8 walks through an example of migrating complex SSIS transformation logic into dbt, using an advanced dbt feature called custom materializations."
slug: custom-materializations-sql-server-dbt

authors: [tyler_rouze,john_barcheski,alexander_lecocq]

tags: [dbt tutorials, analytics craft]
hide_table_of_contents: false

date: 2023-06-01
is_featured: true
---

At [Analytics8](https://www.analytics8.com/), we engage with clients from numerous backgrounds and skillsets. One of our core values is meeting folks where they're at and it's partly why we love working with dbt so much, it enables non-engineers to work like one!

Something we commonly run into these days, is working with people coming from a SQL server background. We're not talking about using dbt with SQL server today, but would rather like to have a discussion about working with dbt to fit into the mental model that is common to come to dbt (and your MPP database of choice) with, and how we might be able to take that background knowledge and make it work within this new dbt paradigm.

If you yourself are reading this post having worked extensively in SQL server in a past life, you likely have seen or developed a SQL pipeline that does something like this:

![insert image here]()

It is not necessarily important that you understand what is happening here, this is just one of the many complex SQL transformation pipelines we have ran into on projects, but what is important to understand is that it's very common to develop custom incremental logic on SQL server. In the days before tools/features like `dbt test`, we had to ensure relational integrity through the use of SQL tables, perhaps stored procedures as well. And so what might have precipitated that necessity, is a pipeline written entirely in SQL on your database system that would move data through a process, determining whether that data passes quality checks, ensuring it has relational integrity to downstream tables it will be joined to, and storing records that don't pass somewhere that we can try again with the next run.

In a way, this mental model was brought to dbt through the use of tests, and [storing failures](https://docs.getdbt.com/reference/resource-configs/store_failures), but we've found it common that the custom logic implemented above can present issues when trying to fit it into dbt's recommended structure, which we will freely admit works best on greenfield projects rather than migrations. For example, I may want to reference my stored failures in another model and perform a retry right away, or manage the records that land in my retry/stored failures table. Connecting each of these logic steps can sometimes be a challenge using out-of-the-box basic dbt features.

And so, what usually happens is a team will develop a complex network of pre- and post-hooks to execute logic in a step by step manner, and while this can work, it does make management more difficult and necessitates the need to build model files for tables that are largely ephemeral in nature.

As a result, we've begun to recommend building [custom materializations](https://docs.getdbt.com/guides/advanced/creating-new-materializations). The reasoning for this is that it gives us the flexibility to fit complex SQL server transformations and stored procedures into dbt in an organized and [DRY](https://docs.getdbt.com/terms/dry) manner. In this post, we will walk through an example of how you might build a custom materialization to emulate SQL server transformation process with retries and relational tests.
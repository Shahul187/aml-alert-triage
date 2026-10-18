# AML alert triage: ranking transaction-monitoring alerts

Banks run rules that flag suspicious transactions for investigators to review. The rules are deliberately cautious, so the queue is huge and about 39 in every 40 alerts turn out to be innocent. This project builds that rules layer on 9.5 million synthetic transactions, then trains a model to rank the alert queue so investigators see the likeliest cases first.

**Result:** reviewing only the top 5% of ranked alerts catches about half of the laundering in the queue, at roughly 26% precision, about 1 in 4 reviewed alerts is real, against 1 in 40 unranked.

**Limitations:** the model only ranks well for accounts with transaction history, and it misses five laundering patterns entirely, including one the simple rules caught perfectly.

**Recommendation:** rank alerts with the model, but keep the rules running as a safety net for the patterns it misses.

[![AML alert triage dashboard](images/dashboard.png)](https://public.tableau.com/views/AMLAlertTriage/AMLAlertTriage)

*Click the image for the interactive version on Tableau Public.*

## The problem

Money laundering is the process of making criminal money look legitimate by moving it through ordinary bank accounts. Banks are legally required to watch for it, and most do so with **transaction monitoring**: a set of rules such as "flag small repeated cash withdrawals" or "flag any transfer over £50,000".

Rules are cheap to run and easy to justify to a regulator, but they are blunt. To avoid missing real cases they flag a lot of normal activity, and every alert must be reviewed by an investigator. Most reviews end with "nothing suspicious", and real cases sit buried in the noise.

This project asks a narrow, practical question: **if investigators can only review part of the queue, which alerts should they look at first?** It does not try to replace the rules. It ranks what they produce, which mirrors how banks actually split the work, with compliance owning the rules and analytics teams improving triage within the queue.

## Data

[SAML-D](https://www.kaggle.com/datasets/berkanoztas/synthetic-transaction-monitoring-dataset-aml) is a synthetic anti-money-laundering dataset published on Kaggle by Berkan Oztas and colleagues. Real bank transaction data is never public, so synthetic data with laundering patterns built in is the standard way to work on this problem openly.

| | |
|---|---|
| Transactions | 9,504,852 |
| Laundering | 9,873 (about 1 in 963) |
| Period | October 2022 to August 2023 (321 days) |
| Laundering patterns | 17 types, from structuring to over-invoicing |
| Geography | About 97% of senders UK-based, with cross-border flows to other countries |

**Working sample:** 9.5 million rows is heavy for free Google Colab, so I kept every laundering transaction and a random 16% of clean ones, giving 1.5 million rows. This raises the laundering rate in the sample to 0.65%, so rates in this README are quoted for the sample and the alert queue rather than the full dataset. The sampling turned out to matter later: it broke the network structure that graph features would have needed (see [Limitations](#limitations)).

**Time-based split:** The model trains on alerts before 1 June 2023 and is tested on alerts from June to August 2023. A random split would let the model learn from the future, which it can never do in practice.

| | Alerts | Laundering |
|---|---|---|
| Train (Oct 2022 – May 2023) | 163,419 | 3,355 |
| Test (Jun – Aug 2023) | 56,544 | 1,416 |

The laundering label and pattern type are used only to train and evaluate the model, never as inputs.

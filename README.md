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

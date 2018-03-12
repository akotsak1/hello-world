# hello-world
Just another repository


***This is the beginning of a Zuora Library of qeuries!

*****This is ALL IN CHURN-- The first query above the UNION statement calculates downgrades, the query below that calculates cancellations.

select a.inc_inception_id__c, max(s.version),
  (CASE when sum(CASE when rp.amendmenttype='NewProduct' THEN rc.MRR END)-sum(CASE when rp.amendmenttype='RemoveProduct' THEN rc.MRR END) IS NULL THEN sum(CASE when rp.amendmenttype='RemoveProduct' THEN rc.MRR END)*-1
   WHEN sum(CASE when rp.amendmenttype='NewProduct' THEN rc.MRR END)-sum(CASE when rp.amendmenttype='RemoveProduct' THEN rc.MRR END) IS NOT NULL THEN sum(CASE when rp.amendmenttype='NewProduct' THEN rc.MRR END)-sum(CASE when rp.amendmenttype='RemoveProduct' THEN rc.MRR END) END)*-1 as Delta_MRR
from zuora.zuora_account a
inner join zuora.zuora_subscription s on (a.id = s.accountid)
inner join zuora.zuora_rate_plan rp on (s.id = rp.subscriptionid)
inner join zuora.zuora_rate_plan_charge rc on (rp.id = rc.rateplanid)
where (s.status='Active' OR s.status='Cancelled') AND ((rc.effectiveenddate >= '2018-03-01' AND rc.effectiveenddate <= '2018-03-31') OR (rc.effectivestartdate >= '2018-03-01' AND rc.effectivestartdate <= '2018-03-31'))
group by s.version, a.inc_inception_id__c
having  (CASE when sum(CASE when rp.amendmenttype='NewProduct' THEN rc.MRR END)-sum(CASE when rp.amendmenttype='RemoveProduct' THEN rc.MRR END) IS NULL THEN sum(CASE when rp.amendmenttype='RemoveProduct' THEN rc.MRR END)*-1
   WHEN sum(CASE when rp.amendmenttype='NewProduct' THEN rc.MRR END)-sum(CASE when rp.amendmenttype='RemoveProduct' THEN rc.MRR END) IS NOT NULL THEN sum(CASE when rp.amendmenttype='NewProduct' THEN rc.MRR END)-sum(CASE when rp.amendmenttype='RemoveProduct' THEN rc.MRR END) END) < '0'
UNION
select a.inc_inception_id__c, max(s.version),
  sum(rc.mrr) as Cancellations
from zuora.zuora_account a
inner join zuora.zuora_subscription s on (a.id = s.accountid)
inner join zuora.zuora_rate_plan rp on (s.id = rp.subscriptionid)
inner join zuora.zuora_rate_plan_charge rc on (rp.id = rc.rateplanid)
where s.status='Cancelled' AND (rp.amendmenttype IS NULL OR rp.amendmenttype='NewProduct') AND (rc.effectiveenddate >= '2018-03-01' AND rc.effectiveenddate <= '2018-03-31')
group by a.inc_inception_id__c

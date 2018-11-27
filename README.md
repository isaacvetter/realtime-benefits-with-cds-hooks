# realtime-benefits-with-cds-hooks

CMS Proposed Rule 4180 (CMS-4180-P;
see [announcement](https://www.cms.gov/newsroom/fact-sheets/contract-year-cy-2020-medicare-advantage-and-part-d-drug-pricing-proposed-rule-cms-4180-p);
see [full text](https://s3.amazonaws.com/public-inspection.federalregister.gov/2018-25945.pdf))
introduces a functional requirement for e-prescribing software to surface real-time pharmacy benefit information into the clinical workflow.
Under this proposal, Medicare Part D plans would expose data or offer a tool to ensure that clinicans could review coverage, cost, medication alternatives,
and prior auth requirements at the point of care, while writing a prescription.

I'd label this proposal as a "functional requirement" because it says something about what the systems needs to do, without
specifying the technical details (standards, protocols, etc) behind the required functionality. Proposing a functional requirement
is a "safe" way to stimulate experimentation with multiple technical approaches -- safe because it avoids trying to over-specify
the technical details, instead allowing market forces to drive innovation. A commonly cited risk is that
each Part D plan could enable this functionality through a different (technically incompatible) approach, meaning (in the worst case)
[~1500 plans](https://q1medicare.com/PartD-History-MedicarePartD-ProgramPDP.php) producing tools that need to integrate with
[~200 e-prescribing systems](https://chpl.healthit.gov/#/search), which would mean 300,000 (!) pairwise integrations. In practice,
things usually aren't nearly this bad; a few predominant approaches emerge and we see market consolidation around them. And 
enterprising broker-type solutions emerge as middleware to bridge across these and offer out-of-the-box connectivity. Still,
an early, open dialog can lead to faster, stronger consolidation, as we saw with the [Argonaut FHIR Profiles for clincial data access](http://www.fhir.org/guides/argonaut/r2/).

CMS explicitly raises the issue of standards in their rationale:

> Because we believe that there currently are no industry-wide electronic standards for RTBTs,
> we are proposing that each Part D plan implement at least one RTBT of its choosing that is
> capable of integrating with prescribers’ e-Rx and EMR systems to provide prescribers who service 
> its beneficiaries complete, accurate, timely and clinically appropriate patient-specific real-time 
> formulary and benefit (F&B) information (including cost, formulary alternatives and utilization 
> management requirements) by January 1, 2020.

Here, I'd like to explore how FHIR and CDS Hooks can enable realtime benefits information to seamlessly
flow into the e-prescribing workflow. As a quick introduction, [CDS Hooks](https://cds-hooks.org/) is an
emerging vendor-neutral standard for integrating external decision support tools into the clinical workflow.
A key use case is to inject decusion support into an e-prescribing session, where the support comes in the form
of "cards" produced by an external service. These cards can have basic information like pricing; they can also
include suggestions for alternative prescriptions or an entry-point into a prior authorization workflow.

Let's review how each of these capabilities can apply in turn, to satisfy the CMS Real-time Benefit Tool requirements.

## The Actors

* Part D plan, in the role of **CDS Hooks Service**. The patient's Part D plan hosts a CDS Hooks compliant endpoint
  to generate advice in near-real-time (which is to say, roughly, under half a second), returning a set of cards for
  the clinician to review.
  
* E-prescribing system, in the role of **CDS Hooks EHR**. The e-prescribing tool, often an EHR-integrated tool,
  calls out to the Part D plan during an e-prescribing session, displaying any returned cards for the clinican
  to review.

In each case, the entry point into the CDS Hooks workflow is an in-progress prescription -- one that a clinician
is writing but hasn't yet completed. The goal is to intervene early, with the full context of the e-prescribing
session so that advice can be tailored to the patient, the drug, and the dosage.

Rather than showing full FHIR Payloads here, I'll use a breezy protocol summary in plain English text. If you want 
to see the full details of what goes back and forth over the wire, check out the [CDS Hooks Sandbox](http://sandbox.cds-hooks.org).


## 1. Sharing information about cost

The Part D plan can return an "information card" that shows pricing details / estimates.

```
EHR:          Hey, Part D Plan! Dr. Mandel is writing a prescription for Sue Smith.
              It's Toprol XL, 25mg daily. Any advice?

Part D Plan:  Yes, please tell Dr. Mandel this is a Tier 3 drug.
              Sue's co-pay is $30/month.
```

## 2. Suggesting an alternative drug
The Part D plan can return a "suggestion card" that shows alternatives.

```
EHR:          Hey, Part D Plan! Dr. Mandel is writing a prescription for Sue Smith.
              It's Toprol XL, 25mg daily. Any advice?

Part D Plan:  Yes, please tell Dr. Mandel this is a Tier 3 drug.
              Offer a suggestion to prescribe generic Metoprolol ER instead.
              It's Tier 1, so Sue will save $27/month.
```


## 3. Entering into a prior auth workflow
For a drug that requires prior auth, the Part D Plan can return a SMART App Link to begin the process automatically.

```
EHR:          Hey, Part D Plan! Dr. Mandel is writing a prescription for Sue Smith.
              It's nitrofurantoin 100mg four times daily. Any advice?

Part D Plan:  Yes, please tell Dr. Mandel this is a Tier 4 drug requiring prior auth.
              Here's a deep link to our prior auth app, if he wants to get started
              right away on the request.
```

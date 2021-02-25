### Profile Base

Most MSH profiles are based on US Core profiles defined in the [US Core Implementation Guide (v3.1.1)](http://hl7.org/fhir/us/core/index.html). For example, [USVeteran] is based on the [US Core Patient][USCorePatientProfile] profile. Because of the way profiles work in FHIR, any resource that validates against an MSH profile that is based a US Core profile will automatically be in compliance with the US Core profile.

Where US Core does not provide an appropriate base profile, MSH profiles FHIR resources.  

| Profile | Based on US Core?  | Immediate Parent Profile |
|---------|--------------------|--------------------------|
| [Combat Episode][CombatEpisode]| no | Observation |
| [Military Occupation][MilitaryOccupation] | no | Observation |
| [Military Service Episode][MilitaryServiceEpisode] | no | Observation |
| [Past Or Present Job][PastOrPresentJob] | no | Observation |
| [US Veteran][USVeteran] | no | Observation |
{: .grid }

### Conformance to MSH Profiles

Each MSH profile expresses requirements and expectations for FHIR instances in terms of structural constraints and terminology bindings. If an instance is required to conform with an MSH profile, it MUST [validate](https://www.hl7.org/fhir/validation.html) against that profile. Only certain FHIR instances associated with an [MSH Patient](conformance-patients.html) carry MSH conformance expectations.

#### Data Sender Expectations

Each MSH profile has a conformance statement describing what data or FHIR instances MUST or SHOULD conform to it. For example, in [PrimaryCancerCondition], the conformance requirements are expressed in two parts:

1. Any Condition resource associated with an [MSH Patient](conformance-patients.html) whose `Condition.code` is in the value set `[PrimaryOrUncertainBehaviorCancerDisorderVS]` MUST conform to the profile.
2. Any resource instance that would reasonably be expected to conform to the profile SHOULD conform to the profile.

The second statement is intended to discourage an MSH Data Sender from creating different representation for data that *should* fall into the scope of MSH. Compliance to this kind of condition is admittedly difficult to enforce, so it is expressed as a SHOULD.

#### Data Receiver Expectations

An MSH Data Receiver SHOULD perform validation on instances it receives. The Receiver first of all needs to identify which profile to use for validation. There four ways to identify the correct profile:

1. The instance is received in response to a [profile search](https://www.hl7.org/fhir/search.html#profile), so the validating profile is known in advance.
2. The instance self-identifies using `meta.profile`. Every Data Sender SHOULD populate this element.
3. The Data Receiver can examine the contents of the instance to associate it with a profile (in particular, by looking for identifying code or category).
4. The Data Receiver can iteratively attempt to validate the instance against each of the Data Receiver's supported profiles.

If an instance fails validation, the Receiver may reject the instance.

### Element-Level Expectations

#### Sender and Receiver Expectations

For every element that is [required](#required-elements) and/or carries a [Must Support obligation](#must-support-obligations) (MS):

* MSH Data Senders SHALL be capable of populating the element, provided the Sender supports the profile (as indicated by its CapabilityStatement).
* If the Sender lacks the data necessary to populate the element:
  * If the element is required, the [US Core rules on missing data](http://hl7.org/fhir/us/core/general-guidance.html#missing-data) MUST be followed.
  * If the element is not required (but is MS), the element SHOULD be entirely omitted. If there is a specific reason the data is missing, a data absent reason MAY be substituted.
  * Senders MUST NOT substitute a nonsense or filler value just to satisfy MS or cardinality requirement.
* MSH Data Receivers SHALL be capable of meaningfully processing MS elements, provided the Receiver supports the profile. Depending on context, "meaningfully process" might mean displaying the data element for human use, reacting to it, or storing it for other purposes.

#### Required Elements

An MSH data element is **required** if any of the following criteria are met:

* The element is a top-level element (a first-level property of the resource) and its minimum cardinality is > 0 in the profile.
* The element not a top-level element (a second-level property or below), its minimum cardinality is > 0, and all elements directly containing that element have minimum cardinality > 0 in the profile.
* The element is not a top-level element, its minimum cardinality is > 0, and its immediate higher-level containing element exists in an _instance_ attempting to conform to the profile.

In other words, a data element may be `1..1`, but if it is contained by an optional element, then it is not required unless its containing element is actually present.

#### Must Support Obligations

Interpretation of MS is not straightforward, as there is a difference between a MS *obligation* and a MS *flag*. The MS *flag* is the red S displayed on profile pages: <span style="padding-left: 3px; padding-right: 3px; color: white; background-color: red" >S</span>. A MS *obligation* means the element must be treated as described in [Sender and Receiver Expectations](#sender-and-receiver-expectations). Significantly, an MS *flag* (<span style="padding-left: 3px; padding-right: 3px; color: white; background-color: red" >S</span>) does not necessarily imply an MS *obligation*, and MS *obligations* may be attached to elements lacking MS *flags*.

To see which elements have MS flags, consult the "Snapshot Table" view of the profile. The "Differential Table" view hides MS flags inherited from the parent profile. The "Snapshot Table (Must Support)" view reflects the IG Publisher's interpretation of how MS flags translate to MS obligations, which may or may not coincide with the US Core/MSH interpretation.

The following rules apply in MSH:

* A profile without a MS flag does not have to be supported [^1]. A participant MUST declare support for optional profiles in its CapabilityStatement.
* Any MS flag or flags on an unsupported profile (as stated in participant's CapabilityStatement) do not carry MS obligations.
* An element with an MS flag does not carry an MS obligation if it is nested and any one of the elements directly containing that element lacks an MS flag. However, if the participant *elects* to support the unflagged element or elements in that hierarchy, the elements below the gap then carry an MS obligation.
* An element with an MS flag whose cardinality is 0..0 does not carry an MS obligation [^2].
* A [required element](#required-elements) carries a MS obligation on the part of a Data Sender, regardless of whether that element has an MS flag. 
* A [required element](#required-elements) without an MS flag does not carry an MS obligation for the Data Receiver [^3].

The following explains how to interpret MS flags in certain cases involving complex elements. Requirements for Senders and Receivers for elements with MS obligations are [defined above](#sender-and-receiver-expectations). For the sake of completeness, this table covers certain cases not seen in MSH.

| # | MS-Flagged Element  | MSH Data Sender (Server) Obligation  | MSH Data Receiver (Client) Obligation | Example |
|---|--------------|--------------|---------------|---|
| 1 | Element is part of a profile that is not supported | No obligation to support | same |  |
| 2 | Element is a nested (child) element and there is no MS flag on its parent element | Must be supported only if the sender/receiver chooses to support the parent element | same | [US Core Patient version 3.2](http://hl7.org/fhir/us/core/3.2.0/StructureDefinition-us-core-patient.html) Patient.telecom.system |
| 3 | Element is a [complex data type](https://www.hl7.org/fhir/datatypes.html#complex) (such as CodeableConcept) with no MS flag on any immediate sub-element  | [MUST support at least one sub-element](http://hl7.org/fhir/us/core/3.2.0/conformance-expectations.html#must-support---complex-elements), and SHOULD support every sub-element the sender has data for | MUST support every sub-element (since the Receiver cannot anticipate which sub-elements might be populated) | [PrimaryCancerCondition.code][PrimaryCancerCondition] |
| 4 | Element is a [complex data type](https://www.hl7.org/fhir/datatypes.html#complex) with an MS flag on one or more immediate sub-elements  | MUST support [only the sub-elements that are explicitly flagged](http://hl7.org/fhir/us/core/3.2.0/conformance-expectations.html#must-support---complex-elements) | same | [CancerPatient.name][CancerPatient] |
| 5 | Element is a [choice \[x\] type](https://www.hl7.org/fhir/stu3/formats.html#choice) with no MS flag on any choice | MUST support at least one datatype choice, and SHOULD populate every datatype for which the server might possess data | MUST support **all** datatype choices (since the Receiver cannot anticipate which sub-elements might be populated)| [CancerPatient.deceased\[x\]][CancerPatient] |
| 6 | Element is a [choice \[x\] type](https://www.hl7.org/fhir/stu3/formats.html#choice) with an MS flag on one or more choice | MUST support only the datatype choice(s) that are explicitly flagged | same |  [US Core Laboratory Result Observation Profile version 3.2](http://hl7.org/fhir/us/core/3.2.0/StructureDefinition-us-core-observation-lab.html) Observation.value[x] |
| 7 | Element is a [Reference() data type](https://www.hl7.org/fhir/references.html#2.3.0) with no MS flag on any referenced resource or profile | MUST support all resources or profiles in the reference that are in Sender's capability statement | MUST support all resources or profiles in the reference unless outside the scope of the Receiver's capability statement | [Tumor.extension\[mcode-condition-related\].value\[x\]][Tumor] |
| 8 | Element is a [Reference() data type](https://www.hl7.org/fhir/references.html#2.3.0) with an MS flag on one or more of the referenced types | MUST support only the resources or profiles in the reference that are explicitly flagged, and only if they are in the Sender's capability statement | MUST support only the resources or profiles in the reference that are explicitly flagged, and only if they are in the Receiver's capability statement | [US Core DocumentReference Profile version 3.2](http://hl7.org/fhir/us/core/3.2.0/StructureDefinition-us-core-documentreference.html) DocumentReference.author
| 9 | Element is a [backbone data type](https://www.hl7.org/fhir/backboneelement.html#2.29.0) | No support expectation on sub-elements unless they are explicitly flagged  | same |
| 10 | Element is an [array that is sliced](https://www.hl7.org/fhir/profiling.html#slicing), with no MS flag on any slice | MUST support populating the array, but [no obligation to support any particular slice](https://confluence.hl7.org/pages/viewpage.action?pageId=35718826#GuidetoDesigningResources-HowdoIinterpret'MustSupport'withrespecttoslicing?) | MUST support arbitrary array contents, including any and all populated slices |
| 11 | Element is an [array that is sliced](https://www.hl7.org/fhir/profiling.html#slicing), with MS flags on one or more slices | MUST support only the slices that have MS flags | same | [TNMStageGroup].hasMember |
| 12 | Element that has MS flag is a slice and the containing array does not have a MS flag | No support expectation for the slice | same | [US Core Patient Profile version 3.1.1](http://hl7.org/fhir/us/core/StructureDefinition-us-core-patient.html) Patient.extension:us-core-race (because Patient.extension is not MS)
{: .grid }

#### Non-Must Support Elements

Data elements in MSH that *do not have* MS obligations still MAY be supported. If an element is supported, whether it has a MS flag or not, the profile must be interpreted as if the MS flag were present. For example, `TumorMarkerTest.performer` does not have an MS flag, but a data receiver may implement the capability to display it. In such a case, by virtue of the fact this element is a Reference() data type with no MS flag on any referenced resource or profile (case #7 above), the receiver would be obligated to support all resources or profiles in the Reference unless outside the scope of the Receiver's capability statement, namely, Practitioner, PractitionerRole, Organization, CareTeam, Patient, and RelatedPerson.

[^1]: Although not common practice, profiles can have MS flags at the very top level (see [CancerPatient] for example).

[^2]: When inheriting from another profile, it is possible to set the upper cardinality to zero on an element that was MS in the parent profile. For example, you could inherit from US Core Patient, but forbid the patient’s name for privacy reasons.  In this case, neither Sender nor Receiver are expected to populate or support the element – in fact, it would be an error if the element were present.

[^3]: An example is a Receiver that does not meaningfully process a required element even though it was populated by the Sender.

{% include markdown-link-references.md %}
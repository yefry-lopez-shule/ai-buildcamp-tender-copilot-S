# Pharma Tender Matcher

Decides, line by line, whether a pharmaceutical company's portfolio technically satisfies what a public health customer requests in a tender. It replaces manual line-by-line product checking so the tender team can focus on competitive pricing. Business decisions such as registration status, pricing, or supply are outside this context.

## Language

### Portfolio

**Product**:
A formulation defined by its active principles, strength, dosage form, and route, independent of how it is packaged.
_Avoid_: Drug, medicine (when meaning the formulation)

**Presentation**:
One concrete, sellable packaging of a Product: a specific Container Type, Fill Volume, and Units per Pack. The Presentation is the unit matched against a tender line.
_Avoid_: SKU, item, package, product (when meaning the sellable unit)

**Dispensing Unit**:
The single container a patient dose is drawn from, such as one vial, ampoule, or prefilled syringe.
_Avoid_: Unit (alone), piece

**Fill Volume**:
The amount of content inside one Dispensing Unit, such as 4 mL in a vial.
_Avoid_: Pack size, container size

**Units per Pack**:
How many Dispensing Units come in one sellable pack; the conversion factor between tender quantities and packs shipped.
_Avoid_: Pack size, pack conversion

**Active Principle**:
A pharmacologically active ingredient of a Product. A Product may have several, such as piperacillin and tazobactam.
_Avoid_: Active ingredient, API, molecule, principio activo (in English text)

**Reference Brand**:
The commercial brand name a Product's Active Principles are commonly known by, such as Xylocaine for lidocaine. A Tender Line that names a Reference Brand is requesting the Active Principle, not that brand.
_Avoid_: Brand (alone), trade name, marca (in English text)

**Biosimilar**:
A biologic Product highly similar to a reference biologic and sharing its Active Principle name; it is matched like any other Product with that Active Principle.
_Avoid_: Generic (for biologics), copy

**Salt**:
The salt form an Active Principle is supplied as, such as hydrochloride. When a Tender Line names no Salt, the Salt is Not Requested; when it names one, it must match.
_Avoid_: Sal (in English text), ester (unless it is one)

**Strength**:
The amount of each Active Principle in a Presentation, expressed as concentration (such as 25 mg/mL), as total amount per Dispensing Unit (such as 100 mg), or both.
_Avoid_: Dose, potency

**Unit Family**:
A group of units that convert into each other exactly, such as mass (g, mg, mcg) and volume (L, mL). Units outside a family, such as IU, mEq, or % w/w, are compared only as written and never converted.

**Strength Basis**:
Whether a Strength is expressed as the Salt or as the free base (such as "vancomycin hydrochloride equivalent to 1 g vancomycin"). Strengths on different bases are never converted into each other automatically.

**Container Type**:
The kind of Dispensing Unit: Vial, Ampoule, Prefilled Syringe, and others. Canonical names are English; Spanish tender wording is resolved to them. Different Container Types are never interchangeable.

**Vial**:
A glass or plastic container closed with a rubber stopper and seal, from which doses are drawn by needle.
_Avoid_: Frasco ampolla, frasco vial, FA (Spanish source wording that means Vial)

**Ampoule**:
A sealed glass container that is broken open for a single use.
_Avoid_: Ampolla (Spanish source wording), vial

**Prefilled Syringe**:
A syringe supplied already filled with the dose.
_Avoid_: Jeringa prellenada (Spanish source wording), PFS

**Pen**:
A prefilled injection device holding one or more doses, such as an insulin pen.
_Avoid_: Pluma, pluma prellenada (Spanish source wording)

**Bag**:
A flexible container for infusion, usually holding a ready-to-administer solution.
_Avoid_: Bolsa (Spanish source wording), IV bag

**Route**:
The way a Presentation is administered, such as intravenous, intramuscular, or subcutaneous. A Presentation may be labeled for several Routes and covers a Tender Line if the requested Route is among them.
_Avoid_: Vía (Spanish source wording); abbreviations such as IV and IM are source synonyms, not separate Routes

**Physical Form**:
Whether the content of a Dispensing Unit is a solid (such as a lyophilized powder) or a solution.
_Avoid_: Presentation (when meaning powder versus solution)

**Pack size**:
An ambiguous source-text term that may mean either Fill Volume or Units per Pack; it is always resolved into one of those and never used on its own.

### Tenders

**Tender Document**:
The file the tender team provides listing a tender's Tender Lines, in whatever form the customer published it (spreadsheet, PDF, or text). The Recommendation is returned as the same Tender Lines with the matching results added.
_Avoid_: Tender file, input

**Tender Line**:
One requested item in a tender, described by the customer in free text. The customer's description is the only basis for a recommendation.
_Avoid_: Item, renglón (in English text), request

### Matching

**Verdict**:
The outcome of comparing one Tender Line against the portfolio: Exact, Equivalent, No Match, or Needs Review, always with a reason.
_Avoid_: Score, result

**Exact**:
A Verdict where a Presentation matches everything the Tender Line specifies, field by field.
_Avoid_: Perfect match, identical

**Equivalent**:
A Verdict where the Tender Line is generic (leaves fields open) and the Presentation satisfies everything it does specify; several Presentations may be Equivalent for the same line, such as 2 mL, 4 mL, and 6 mL vials of the same strength.
_Avoid_: Similar, substitute, alternative

**No Match**:
A Verdict where no Presentation in the portfolio satisfies every field the Tender Line specifies.
_Avoid_: Fail, rejected

**Needs Review**:
A Verdict meaning neither deterministic matching nor the agents could decide with confidence; only these Tender Lines go to a person.
_Avoid_: Pending, unclear, manual

**Escalation**:
Passing a Tender Line to the next resolver when the current one cannot decide with confidence: deterministic matching, then the Matching Agent, then the Reviewer Agent, then a person.

**Matching Agent**:
The agent that resolves Tender Lines deterministic matching could not decide, using portfolio search, unit normalization, and Presentation details.

**Reviewer Agent**:
A second agent that checks the Matching Agent's Verdict on an escalated Tender Line before it is either accepted or sent to a person.
_Avoid_: Judge, second opinion

**Recommendation**:
The system's output for a tender: the Verdicts for all Tender Lines plus the Bid Coverage. It is a draft that the tender team acts on; the system never submits a bid.
_Avoid_: Bid, offer, submission

**Recommended Presentations**:
The Presentations proposed for a Tender Line; there may be several, especially when the Verdict is Equivalent.

**Matched Fields**:
The fields of a Tender Line that the Recommended Presentation satisfies.

**Unmatched Fields**:
The fields of a Tender Line that the Presentation does not satisfy; they are the reason behind a No Match or Needs Review Verdict.

**Confidence**:
A percentage stating how sure the system is of the Verdict on a Tender Line.
_Avoid_: Score, probability, similarity

**Confidence Threshold**:
The minimum Confidence at which a resolver's Verdict is accepted; below it, the Tender Line is escalated.

**Pharma Interested**:
The per-line answer to "should we bid on this line?": True when the Verdict is Exact or Equivalent, False when it is No Match, and Pending Human when it is Needs Review.
_Avoid_: Biddable, eligible, participate

**Historical Awarded Price**:
The price at which a previous tender awarded the same or a comparable Tender Line. It is reported alongside the Verdict as input for the tender team's pricing and never affects the Verdict.
_Avoid_: Reference price, last price

**Bid Coverage**:
The share of a tender's Tender Lines whose Pharma Interested is True. Lines that are Pending Human are reported separately.
_Avoid_: Win rate, coverage (alone)

**Defining Field**:
A field that identifies which Presentation is meant, such as Active Principles, Strength, Fill Volume, Container Type, Route, and Physical Form. A Tender Line that leaves a Defining Field open is generic. Salt is not a Defining Field: an unnamed Salt is Not Requested.

**Additional Attribute**:
A field that describes a Presentation without identifying it, such as preservative-free. When the Tender Line does not request it, it does not affect the Verdict.

**Not Requested**:
The state of a field the Tender Line does not specify. It is a fact about the customer's request, not about the portfolio.
_Avoid_: Missing, null, blank

**Unknown**:
The state of a field whose value is absent from the portfolio record for a Presentation. It is a fact about the portfolio, not about the request.
_Avoid_: Missing, null, blank

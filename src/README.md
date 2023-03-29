# Automated testing of conformance rules

The contents of this folder are used for automated testing of the conformance rules specified in the IG.

The IG Publisher tests conformance of example resource instances provided as part of the IG, but build times can creep above 5 minutes for a large IG, which makes iterative testing of conformance rules (invariants in particular) very cumbersome.

Additionally, the IG Publisher does not provide an easy mechanism for testing conformance of resource instances that are not examples. You might want to do this to test failure of a conformance rule (e.g., provide a resource instance designed to fail an invariant to ensure the invariant works as intended).

The contents of this folder allow for easy testing of a defined set of FHIR resource instances against the IG in two ways:

1. `src/test/resources/tests.csv`: List of resource instances that should fully validate
2. `src/test/resources/tests_errors.csv`: List of resource instances (typically stored in `src/test/resources/` as JSON files) that _should_ produce a specific validation failure.

If you need to include additional external FHIR dependencies, add them to `src/test/java/mcode/testing/ProfileTest.java`.

## Running

At the command prompt in the root folder of the IG, run:

- Mac/Linux: `./mvnw test`
- Windows: `mvnw test`

You **do not** need to re-run SUSHI or the IG Publisher before running the tests: the above commands run SUSHI as a preflight step.

## Example testing strategy

The invariant `must-have-focus-or-specimen-invariant` (defined for the tumor size profile) has the following contents:

    Invariant: must-have-focus-or-specimen-invariant
    Description: "Either `focus` OR `specimen` MUST be populated"
    Expression: "(focus.exists() or specimen.exists()) and (focus.exists() and specimen.exists()).not()"
    Severity: #error

A comprehensive testing strategy for this invariant would be as follows:

1. Test the `Observation-tumor-size-pathology` resource instance generated by FSH as an IG example, which includes `focus` by including the following line in `tests.csv`:

        fsh-generated/resources/Observation-tumor-size-pathology.json,http://hl7.org/fhir/uv/mcode/StructureDefinition/mcode-tumor-size

2. Define a resource instance that includes `specimen` but not `focus`, and test for successful validation in `tests.csv`. If you don't want to have an official example in the IG of this, you can put a JSON resource instance in `src/test/resources/`.

        src/test/resources/must-have-focus-or-specimen-invariant_specimen-only.json,http://hl7.org/fhir/uv/mcode/StructureDefinition/mcode-tumor-size

3. Define a resource instance with **both** `focus` and `specimen`, and test for a validation failure in `tests_errors.csv`:

        src/test/resources/must-have-focus-or-specimen-invariant_focus-and-specimen.json,http://hl7.org/fhir/uv/mcode/StructureDefinition/mcode-tumor-size,ERROR,Observation,must-have-focus-or-specimen-invariant

4. Define a resource instance with **neither** `focus` nor `specimen`, and test for a validation failure in `tests_errors.csv`:

        src/test/resources/must-have-focus-or-specimen-invariant_neither.json,http://hl7.org/fhir/uv/mcode/StructureDefinition/mcode-tumor-size,ERROR,Observation,must-have-focus-or-specimen-invariant

This example is in fact implemented -- see below.

----

## Implemented tests

Because it's not possible to include comments in the `.csv` files that define the tests, it is **strongly recommended** to briefly describe implemented test strategies below -- otherwise it will be difficult to follow what is happening in the CSV files.

### `must-have-focus-or-specimen-invariant`

- Successful validation expected
    - Includes `focus` only: `fsh-generated/resources/Observation-tumor-size-pathology.json`
    - Includes `specimen` only: `src/test/resources/must-have-focus-or-specimen-invariant_specimen-only.json`
- Errors expected
    - Includes both `focus` and `specimen`: `src/test/resources/must-have-focus-or-specimen-invariant_focus-and-specimen.json`
    - Includes neither: `src/test/resources/must-have-focus-or-specimen-invariant_neither.json`
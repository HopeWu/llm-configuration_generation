# llm-configuration_generation
Use LangChain and LLM to Generate Configuration

When you need to create a configuration file that has thousands of lines, you may want to write a script for that. Here is an example of doing so to write a configuration file that has ~7000 lines. I wrote this script for a real task in a full time job as a data scientist.

### What is the task?
The task is to extend the configuration file for a value parser component that seperates value and its unit, e.g. "220 V" -> `{"value": 220, "unit": "VOLT"}`. A configuration may look like this.
```yaml
  POWER:
    units:
    - allow_fractions: false
      forms:
      - w
      - watts
      - watt
      norm: WATT
    - allow_fractions: false
      forms:
      - kw
      - kilowatt
      - kilowatts
      norm: KILOWATT
    - allow_fractions: false
      forms:
      - mw
      - milliwatt
      - milliwatts
      norm: MILLIWATT
    - allow_fractions: false
      forms:
      - btu
      - btus
      - b.t.u
      - b. t. u.
      - british thermal unit
      norm: BTU
    - allow_fractions: true
      forms:
      - hp
      - horse power
      - horsepower
      - horse-power
      norm: HP
    value_types:
    - single_value
    - alternatives
    - range
```
Specifically, the task is that given the name of a property, e.g. POWER, create a configuration like above and it involves configuring for 300 properties. I have gpt-35-turbo-instruct model and carried out two ways of doing this. The first approach didn't work well but the other one certainly worked as a charm.

### How I solved it?
The first one is merely few-shot-prompting. To be more specific, let llm generate a configuration with a property name when examples are provided. This end-to-end generation doesn't give enough patterns/instructions for the llm to follow given this particular task. Therefore, the next approach divides the big task into smaller and more specific ones, which are easier for the llm to tackle one by one. LangChain comes handy here with the flexibility of building multiple chains. The results are delightful. It's almost 100% accurate.
```yaml
  PROCESSING_(MELT)_TEMP:
    implicit_unit: "\xB0C"
    unbound_range_support:
      and_over_regex: '>|more than'
      and_under_regex: <|less than
      lower_limit: 0
      upper_limit: infinity
    units:
    - allow_fractions: true
      forms:
      - "\xB0c"
      - "\xB0C"
      - celsius
      - degrees Celsius
      norm: "\xB0C"
    - allow_fractions: true
      forms:
      - kelvin
      - k
      - K
      - "\xB0K"
      norm: K
    - allow_fractions: true
      forms:
      - deg F
      - "\xB0F"
      - "\xB0f"
      - degrees Fahrenheit
      norm: "\xB0F"
    value_types:
    - single_value
    - range
```

In the second approach, the LLM is used for two purposes. One is to get several units for a given property, e.g. K, °F, °C for PROCESSING_(MELT)_TEMP. The other is to get several forms of a given unit, e.g. kelvin, k, K for K. The assumption is the underlying LLM should have this general knowledge, which is basically true.

### Is LangChain pleasant to use here?
Yes, it is. `LCEL` is utilised whenever possible. Using it is easy and makes the code clean and flexible. Two chains are created accordingly for the two sub tasks mentioned above. Postpocessing are implemented as custom output parsers that are used to deal with cases where LLM is not doing well enough. `RunnableBranch` is used for bypassing LLM in certain cases. `RunnableLambda` is used to chain arbitrary functions into the `chain`s.

### Notes
The code is not runnable by you due to lack of data that is sensitive to share. You can certainly mock it if you want. Some of the output of cells are preserved. 

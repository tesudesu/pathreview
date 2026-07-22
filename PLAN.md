## Solution plan

**Issue:** #146 - PII scrubber fails to redact parenthesized US phone numbers (https://github.com/ascherj/pathreview/issues/146)

### Understand
I expect all 10-digit phone numbers (with an optional 1 country code at the front) in various formats - dashes, parenthesis, dots - to be detected and redacted. Some phone number formats, e.g. 555-123-4567, are redacted as expected. But parenthesized phone numbers, e.g. (555) 123-4567, are not being redacted.

**Root cause:** Bugs in the regex pattern for the "phone_us" type in `safety/pii_scrubber.py`.

### Map
Files I expect to touch:
- `safety/pii_scrubber.py`: Specifically line 15, where the regex for "phone_us" is.
- `tests/unit/test_pii_scrubber.py`: Possibly to add new tests for edge cases.

### Plan
1. Examine `safety/pii_scrubber.py`
2. Notice the PIIScrubber class contains different regex patterns for what should be redacted, a scrub function to replace a detected PII with the string '[REDACTED]', and a detect function that determines if a string matches any of the regex patterns. 
3. The error is likely in the "phone_us" regex pattern: `"phone_us": r"\b(?:\+?1[-.]?)?\(?([0-9]{3})\)?[-.]?([0-9]{3})[-.]?([0-9]{4})\b"`
4. Break down the "phone_us" regex pattern to understand why phone numbers that use the parenthesis format aren't matched.
5. Apply the fix.
6. Possibly add new tests for edge cases.
6. Run the tests to make sure all existing failed tests pass and the fix doesn't make other tests fail. (Note: `test_mixed_pii_and_text` already fails, but it is likely related to another regex pattern and is thus out of the scope of this issue.)

### Inputs & outputs
The fix would directly apply to the "phone_us" regex pattern. After the fix is applied, parenthesized US phone numbers, e.g. (123) 456-7890, should be replaced with "[REDACTED]".

### Risks & unknowns
There could be other unknown bugs in the regex pattern. I am not sure if phone numbers with parenthesis or dashes in unexpected places should be redacted as well, e.g. (5822) 556-990. I'm also unsure if phone numbers only separated by spaces, e.g. 555 123 4567, should be redacted. 

### Edge cases
- 1234567890
- (555)123-4567
- (555) 123 - 4567
- +1 (555) 123-4567
- Possibly separators in unexpected places, e.g. (1234) 567-890
- Leading or trailing separators, e.g. (555) 123-4567.
- Phone numbers at the start or end of the string
- Possibly numbers with fewer or extra digits, e.g (555)123-456
- Possibly numbers with only spaces, e.g. 555 123 4567


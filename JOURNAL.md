## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/146

**Issue title:** PII scrubber fails to redact parenthesized US phone numbers

**Tier:** Tier 1

**Problem summary:**
Phone numbers are supposed to be redacted. But right now, only phone numbers in dashed formats are redacted. Phone numbers in the parenthesized format (555) 123-4567 pass through `scrub()` unredacted, and `detect()` reports no PII for it.

The phone number pattern is in `pii_scrubber.py`. 

The related failing tests are: `test_us_phone_number_redaction`, `test_us_phone_formats`, `test_detect_phone_pii`, and `test_phone_at_start_of_text` in `tests/unit/test_pii_scrubber.py`.

**"Is this right for me?" checklist reasoning:**
I can explain what the issue is asking for. I think that the bug may lie in `pii_scrubber.py`. I know where the tests for phone numbers are located. I know what done looks like - a phone number that uses parenthesis should be redacted. A tier 1 issue is a good fit for me, who has not pushed code to open source before. The scope of this issue is fairly confined and so should be realistic for a 3-4 week project. I haven't noticed any blockers yet. The issue is still open.

**Branch name:** fix/146-redact-phone-numbers-error

**Setup confirmation:** App runs locally at localhost:5173

**Cohort ledger:** Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [39d88cd](https://github.com/tesudesu/pathreview/commit/39d88cd)

**Reproduction summary:**
I reproduced this issue in two ways, first by running the test at `tests/unit/test_pii_scrubber.py`. The failed tests related to this issue are `test_us_phone_number_redaction`, `test_us_phone_formats`, `test_detect_phone_pii`, and `test_phone_at_start_of_text`.

I also invoked the scrubber directly in the terminal, running

```python
from safety.pii_scrubber import PIIScrubber
s = PIIScrubber()
t = 'Call me at (555) 123-4567 or 555-123-4567'
print('scrub :', s.scrub(t))
print('detect:', [d['type'] for d in s.detect(t)])
```

The output is 

```
scrub : Call me at (555) 123-4567 or [REDACTED]
detect: ['phone_us']
```

As you can see, only the first 555-123-4567 phone number is detected as 'phone_us' and redacted. The second one with parenthesis, (555) 123-4567, is not. 

**PLAN.md link:** [PLAN.md](https://github.com/tesudesu/pathreview/blob/fix/146-redact-phone-numbers-error/PLAN.md)

**Blockers or open questions:**
None

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
All steps in PLAN.md are essentially done. The basic fix is implemented and new tests are written.

**Next steps:**
I will look more closely at the new tests and consider whether to adjust the implementation to cover more edge cases. 

**Blockers:**


---

### Check-in 2 (end of week)

**PR link:** https://github.com/tesudesu/pathreview/pull/1

**Branch:** `fix/146-redact-phone-numbers-error`

**What you built:**
The original "phone_us" regex pattern did not allow whitespace separators, so formats like "(123) 456-7890" (space after the parenthesis) were not caught. The original pattern also did not catch the leading parenthesis. Both bugs have now been fixed. 

**Tests added or updated:**
Added 4 new tests to `test_pii_scrubber.py`:

`test_phone_us_formats_redacted`: Checks that a US phone number is fully redacted. 

`test_phone_us_no_orphaned_prefix`: Makes sure that prefixes like the leading parenthesis or a plus sign are captured with the phone number.

`test_phone_us_trailing_punctuation_preserved`: If the phone number is followed by a full stop, e.g. `(123) 456-7890.`, ensures that the full stop is not captured along with the phone number. 

`test_phone_us_no_false_positives`: Requires the 3, 3, 4 numbers blocks for the US phone number format, so other numbers are not falsely captured.  

**Self-review confirmation:** [x] make check passes (failures are pre-existing)  [x] make test-unit passes (the `test_mixed_pii_and_text` test failure is pre-existing and unrelated to the "phone_us" regex pattern)

**Draft PR feedback received from:** 
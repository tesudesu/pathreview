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
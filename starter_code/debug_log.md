# Debug log (Part A)

Name: ____________________   Date: ____________________

Fill one block per ticket, in the order you fixed them. Use the method from the lesson: Reproduce, Read,
Isolate, Hypothesise, Test, Fix, Verify. Short answers are fine, but the EVIDENCE line must be something
you actually saw (a pdb command and its output, or a print line).

## BUG-1
- Symptom (what I saw): Running py run_report.py 1 returned 3 instead of the expected 2.5.
- Family (syntax / runtime / logic): Logic
- Hypothesis (before I looked): The function wasn't checking whether the list had an even or odd number of elements.
- Evidence (pdb command + output): p ordered returned [1, 2, 3, 4], p len(ordered) // 2 returned 2, and p ordered[len(ordered) // 2] returned 3.
- Root cause (one sentence): The function always returned a single middle element instead of averaging the two middle values for an even-length list.
- Fix (one sentence): - Fix (one sentence): Added an empty-list check and separate logic for odd and even-sized lists.

- Verified by: `py run_report.py 1` returned 2.5, and `median([])` raised the expected ValueError.

## BUG-2
- Symptom (what I saw): Running `py run_report.py 2` originally returned `[340, 210]` instead of the expected `[340, 210, 150]`.
- Family (syntax / runtime / logic): Logic
- Hypothesis (before I looked): The function was using incorrect list slicing, causing it to return fewer elements than requested.
- Evidence (pdb command + output, or print output): The original run returned `[340, 210]`. Testing the updated function with `n=3` returned `[340, 210, 150]`, with `n=10` returned `[340, 210, 150, 100]`, and with `n=0` returned `[]`.
- Root cause (one sentence): The function used `durations[:n - 1]`, which excluded one requested element, and `.sort()` modified the caller's original list.
- Fix (one sentence): Replaced the in-place sorting with `sorted()` and used `[::-1][:n]` to return the requested number of durations in descending order.
- Verified by: Running `py run_report.py 2` returned `[340, 210, 150]`, and additional tests confirmed correct handling of zero, oversized requests, and preservation of the original list.

## BUG-3
- Symptom (what I saw): Running `py run_report.py 3` crashed with `KeyError: 'login'`.
- Family (syntax / runtime / logic): Runtime
- Hypothesis (before I looked): The program was trying to access a suite in the dictionary before creating an entry for it.
- Evidence (pdb command + output, or print output): `p grouped` returned `{}`, `p result` returned `{'test': 't1', 'suite': 'login', 'status': 'pass'}`, and `p result["suite"]` returned `'login'`.
- Root cause (one sentence): The function tried to increment a status count before initializing the nested dictionary for the suite and status.
- Fix (one sentence): Added checks to initialize missing suite dictionaries and used `.get()` to retrieve and increment status counts, defaulting missing suites to `"unknown"`.
- Verified by: `py run_report.py 3` returned `{'login': {'pass': 2, 'fail': 1}, 'search': {'pass': 1}}`, and a record without a suite was correctly grouped under `"unknown"`.

## BUG-4
- Symptom (what I saw): Running `py run_report.py 4` returned `(['t2', 't9'], ['t2', 't9'])` instead of two separate lists.
- Family (syntax / runtime / logic): Logic
- Hypothesis (before I looked): Both function calls were sharing the same list instead of creating separate lists.
- Evidence (pdb command + output, or print output): The original run returned `(['t2', 't9'], ['t2', 't9'])`. After changing the default argument, the output became `(['t2'], ['t9'])`.
- Root cause (one sentence): Using `flaky=[]` as a default argument caused Python to reuse the same mutable list across function calls.
- Fix (one sentence): Changed the default argument to `flaky=None` and created a new empty list whenever no list was provided.
- Verified by: Running `py run_report.py 4` returned `(['t2'], ['t9'])`, and testing with an existing list confirmed that the caller's list was updated.

## BUG-5
- Symptom (what I saw): Running `py run_report.py 5` crashed with an `UnboundLocalError` when attempting to generate a report ID.
- Family (syntax / runtime / logic): Runtime
- Hypothesis (before I looked): The function was trying to modify `run_counter` without correctly accessing the module-level variable.
- Evidence (pdb command + output, or print output): The traceback pointed to `run_counter += 1` in `next_run_id()` and reported `UnboundLocalError: cannot access local variable 'run_counter' where it is not associated with a value`.
- Root cause (one sentence): Python treated `run_counter` as a local variable because it was assigned inside the function without a `global` declaration.
- Fix (one sentence): Added `global run_counter` before incrementing the module-level counter.
- Verified by: Running `py run_report.py 5` returned `RUN-0001`, and three consecutive calls in one Python process returned `RUN-0001`, `RUN-0002`, and `RUN-0003`.

## BUG-6
- Symptom (what I saw): The original `slowest()` implementation modified the durations list, which could cause the dashboard to display the wrong latest duration.
- Family (syntax / runtime / logic): Logic
- Hypothesis (before I looked): The `slowest()` function was sorting the original list before `summarise_run()` accessed its last element.
- Evidence (pdb command + output, or print output): After fixing BUG-2, `py run_report.py 6` showed `'latest_ms': 150`. An additional test printed `Latest: 150` and `Original list: [100, 340, 210, 150]`.
- Root cause (one sentence): The original `slowest()` function used `.sort()`, which changed the caller's list and affected the value retrieved using `durations[-1]`.
- Fix (one sentence): The BUG-2 fix replaced `.sort()` with `sorted()`, preserving the original list and correcting the dashboard's latest duration.
- Verified by: Running `py run_report.py 6` returned `latest_ms: 150`, and the original durations list remained unchanged.

## BUG-7
- Symptom (what I saw): Running `py run_report.py 7` crashed with `ValueError: median() needs at least one value`.
- Family (syntax / runtime / logic): Runtime
- Hypothesis (before I looked): The summary function was attempting to calculate the median even when the durations list was empty.
- Evidence (pdb command + output, or print output): The traceback showed that `summarise_run()` called `median(durations)`, which raised a `ValueError`. After the fix, the output showed `'median_ms': None, 'slowest_ms': [], 'latest_ms': None`.
- Root cause (one sentence): The summary function did not check whether durations existed before calculating the median or accessing the last element.
- Fix (one sentence): Added conditional expressions to return `None` for median and latest duration when the list is empty.
- Verified by: Running `py run_report.py 7` successfully returned a summary containing `median_ms: None`, `slowest_ms: []`, and `latest_ms: None` without crashing.

## Reflection (3 sentences)
BUG-3 was the hardest to understand because I had to investigate how nested dictionaries were initialized and updated. Using Pdb helped the most because I could inspect variables and understand their values before the program crashed. Next time, I would check the function's requirements and test edge cases earlier to identify problems before they affect other functions.

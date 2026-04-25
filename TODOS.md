# TODOS and Known Issues
# Barclays - Generative AI: Prompt Engineering for Software Developers
#
# Entries are added by /fixes and resolved entries are moved to AUDIT.md.

---

## [RESOLVED] topic_01 plan file has bare --- separators and research URLs not as https:// links - 2026-04-25

**Topic**: topic_01_foundations
**Reported**: 2026-04-25
**Resolved**: 2026-04-25
**Description**: /verify 1 found two issues in plans/topic_01_foundations.md:
  1. 25 bare --- separator lines throughout the plan file (between cell sections). These
     violate the no-bare-separator rule from CLAUDE.md.
  2. Research source URLs in the RESEARCH VALIDATED section were formatted as plain text in
     a table, not as https:// hyperlinks. The verify-research checker found 0 URLs, failing
     the minimum-3-sources check.
**Status**: resolved
**Fix applied**: Removed all 25 bare --- separators using a python3 script that skipped lines
inside code fences. Replaced the RESEARCH VALIDATED table with a numbered list of 7 properly
formatted https:// URLs. Both checks now pass: AI-tells CLEAN, 7 https:// URLs found.

# SpellCheckCustomDictionary — Dev Trial instructions

Feature entry: https://chromestatus.com/feature/6185007701557248
Explainer: https://github.com/Igalia/explainers/tree/main/spell-check-dictionary

  The ```SpellCheckCustomDictionary``` API lets a page add words to a per-document custom dictionary so the spell checker stops flagging them as misspellings.

  Typical use: an editor whose content contains domain vocabulary, product names etc. that the user should not see squiggled when spelling check is enabled.
 
## Enabling the feature

  Enable "**Experimental Web Platform Features**" in ```chrome://flags``` to access the feature for testing or launching Chrome from the command line with the Blink feature enabled.

```  
chrome --enable-blink-features=SpellCheckCustomDictionaryAPI
```
## Prerequisites

  All of the following must hold. If any is missing, ```addWords()``` and ```removeWords()``` are silent no-ops — they don't throw, so a failing prerequisite looks like the API doing nothing.

  1. Chrome Canary or Dev, milestone 153 or newer (developed against 153.0.7983.0).
  2. Desktop — Linux, macOS, Windows, ChromeOS. Android is not part of this trial.
  3. **Secure context**. ```https://``` or ```http://localhost```.
  4. **A Window context**. The interface is ```Exposed=Window```; it is not available in workers.
  5. **Browser spell checking turned on**. *chrome://settings/languages → Spell check → Check for spelling errors enabled, with at least one language selected and its dictionary downloaded*. If spell checking is off, the API returns early and does nothing.
  6. **Text that actually gets spell checked**. A contenteditable element, a ```<textarea>```, or``` <input type="text">```, without ```spellcheck="false"```.

## API surface

  ```
  [Exposed=Window, SecureContext]
  interface SpellCheckCustomDictionary {
    undefined addWords(sequence<DOMString> words);
    undefined removeWords(sequence<DOMString> words);
  };

  partial interface Document {
    [SameObject] readonly attribute SpellCheckCustomDictionary spellCheckCustomDictionary;
  };
  ```
Both methods are synchronous and return undefined. There is no ```getter``` — the word set cannot be read back.

  ## Example
  
Check on the [WPT manual test](third_party/blink/web_tests/wpt_internal/spell-check-custom-dictionary/add-remove-words-manual.html).

  ## Behaviour to be aware of while testing

  These are current, intentional properties of the implementation. They are the parts we would most like feedback on.

  - **Document-scoped and non-persistent**. The word set belongs to the current document and is cleared on navigation. A page must re-add its words after every navigation, including same-origin ones.
  - **Write-only**. There is no way to enumerate or query the current set.
  - **Independent of the user's own dictionary**. Words added through this API do not appear in chrome://settings/editDictionary, are not synced, and do not affect other tabs. Conversely the user's existing custom dictionary continues to apply.
  - **Exact, case-sensitive matching**. A word is suppressed only when the flagged text matches an added entry exactly. Adding "Pikachu" does not suppress "pikachu".
  - **Invalid entries are silently skipped**, matching the browser custom dictionary's validation: empty strings, words with leading or trailing whitespace, and ill-formed UTF-16 (unpaired surrogates). No error is raised.
  - **Limits**. 20,000 words per document, and 128 bytes (UTF-8) per word. Additions past either limit are dropped, and one console warning is logged per document: SpellCheckCustomDictionary: *per-document word limit reached or a word exceeded the maximum length*.

  ## Feedback

  - Spec/design discussion: file an issue on the explainer repository https://github.com/Igalia/explainers.
  - Implementation bugs: https://issues.chromium.org/issues/428005649


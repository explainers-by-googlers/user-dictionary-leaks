# Explainer: Preventing User Dictionary Leaks via ::spelling-error and ::grammar-error CSS Pseudo-Elements

## [Discuss](https://github.com/explainers-by-googlers/user-dictionary-leaks/issues)

## Introduction

The user’s dictionary may contain sensitive information, for example some operating systems import the contents of the user’s address book to assist with the spelling of names/addresses. For this reason, when the `::spelling-error` and `::grammar-error` pseudo elements are applied to a text input field they [cannot be read directly](https://drafts.csswg.org/css-pseudo/#highlight-security) via JavaScript.

## Attack Model

Although direct indicators of the `::spelling-error` and `::grammar-error` cannot be extracted, it’s possible to extract indirect information from browsers without rate limits on the application of these hints. In Chrome and Firefox, it’s possible to have an [autofocused](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/autofocus) text area cycle programmatically through a series of misspelled words, and for the site to monitor indicators of rendering performance to notice when hints are applied. This allows sites (or their third-party embeds) to detect which words are or aren’t in the user’s dictionary, which could leak sensitive information stored there (for example, their contacts' names). Safari already has rate limits in place which only check for and apply hints once per user interaction with the text field (e.g., a key input or click).

## Goals

* Prevent user dictionary data from being extracted without user interaction/consent

## Non-Goals

* Prevent sites from ever gaining any information about the user’s dictionary

## Proposed Solution

We should amend section [3.7. Security Considerations for Highlighting](https://drafts.csswg.org/css-pseudo/#highlight-security) of the CSS Pseudo-Elements web specification to require the following to prevent indirect indicators of the user’s dictionary contents from being leaked:

* Hints must not be applied to a text field that has not had user interaction (an [autofocus](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/autofocus) is insufficient, there must be a click or key press of some kind relative to that field).
* Hints must only be applied once per user interaction (the text cannot be changed programmatically and have hints applied without a click or key press of some kind relative to that field).

Safari is already in full compliance with both points, while Firefox and Chrome are only in partial compliance with the first one (they do count autofocused fields, but don’t apply new hints to fields that aren’t in active focus).

## Alternatives Considered

### Permissions Prompt

We could require explicit user permission before applying hints (either at the site level or even for individual text fields). This would be quite disruptive to current user and site expectations, and would face significant friction in rollout even if it did better secure user data against indirect inferences sites could make.

## Compatibility Concerns

It’s possible that existing websites mostly accessed in Firefox, Chrome, or other browsers with similar spelling/grammar hint behavior, expect hints to appear on autofocus or for programmatically inserted text. That said, a temporary lack of spelling/grammar hints should not be the most significant breakage. The privacy risk we seek to mitigate here is significant enough that some inconvenience on those paths is probably acceptable as long as the user has a clear, direct way to cause hints to be applied (click on or typing into the text field).

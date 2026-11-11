# Learning Media Clearance Policy

Version 3.2 | Effective 2026-10-01 | Owner: Media Operations

## Purpose

Every external video proposed for internal training use must pass a source
identity check and a rights check before it may be added to the approved
catalog.  The queue is the single intake list for a weekly run.

## Source verification

For each queued video, resolve its public source listing and record the live
title and channel (author) shown by the source platform. The platform exposes
listing metadata through its oEmbed endpoint, for example
`https://www.youtube.com/oembed?url=<full video URL>&format=json`; the `title`
and `author_name` fields are the live title and channel.

* If the source listing cannot be resolved, the status is `unavailable`.
* Compare the live channel with the queue's `claimed_channel` after trimming
  surrounding whitespace and ignoring letter case.  If they differ, the status
  is `channel_mismatch`.
* If the channel matches but `rights_status` is anything other than `cleared`,
  the status is `rights_pending`.
* Otherwise the status is `approved`.

## Priority mapping

| Status | Priority |
| --- | --- |
| unavailable | P1 |
| channel_mismatch | P1 |
| rights_pending | P2 |
| approved | none |

## Required actions

For every candidate that is not approved, open exactly one tracking issue in
the team repository.

* Title: `[Clearance] <video_id> - <reason>`
* Reason text: `unavailable source`, `channel mismatch`, or `rights pending`.
* Labels: `clearance-review` plus `priority:P1` or `priority:P2`.
* The issue body must contain these lines somewhere in the text:

```
Video ID: <video_id>
Status: <status>
Claimed channel: <claimed_channel>
Observed channel: <observed channel, or N/A>
Rights status: <rights_status>
Decision: <reason>
```

For every approved candidate, append one item to `catalog/approved_catalog.json`
using the live title, live channel, source URL and the run date as
`approved_date`.  Existing catalog items must not be modified or removed.

## Run record

Publish a run record at `runs/clearance_results_<run_date>.json` with this shape:

```
{
  "run_date": "YYYY-MM-DD",
  "results": [
    {
      "video_id": "...",
      "status": "...",
      "reason": "...",
      "issue_number": 123 or null
    }
  ],
  "summary": {
    "approved": 0,
    "rights_pending": 0,
    "channel_mismatch": 0,
    "unavailable": 0
  }
}
```

## Intake issue

When a weekly intake issue exists for this run, post one completion comment in
this exact form and then close the issue:

`Clearance review completed: approved=<n>, rights_pending=<n>, channel_mismatch=<n>, unavailable=<n>`

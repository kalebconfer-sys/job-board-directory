# Job board directory

Public, free directory of **12,784 job board addresses** across nine
applicant-tracking systems, each one confirmed live against the vendor's own
public API.

**https://kalebconfer-sys.github.io/job-board-directory**

| Platform | Boards |
|---|---|
| Greenhouse | 3,574 |
| Ashby | 2,486 |
| Workable | 1,875 |
| Workday | 1,251 |
| Recruitee | 1,043 |
| Personio | 866 |
| Rippling | 864 |
| SmartRecruiters | 662 |
| Lever | 163 |

## Why

Every one of these platforms serves a company's jobs from a public endpoint, and
every one of them hides the hard part: the address. A Greenhouse token, a Lever
slug and above all a Workday host/tenant/site triple cannot be derived from a
company name. There is no complete free directory of them, so people guess — and
a wrong guess on Workday returns a server error rather than a clear "no such
board".

## How a board gets listed

1. The address is observed in a public crawl or public index.
2. It is called against the vendor's own documented public API.
3. It is listed only if that call returned a live board.
4. Where the vendor publishes its own job count, that published count is shown,
   so it can be checked. Where the vendor publishes none, the number is what was
   observed and is labelled as such.

Nothing here is behind a login and no access control was circumvented. Counts
are timestamped observations, not promises — boards change daily.

## As data

Each platform has a matching Actor that returns the whole directory free, or the
postings from any board you name: https://apify.com/blooming_gator

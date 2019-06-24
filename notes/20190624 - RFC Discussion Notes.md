# RFC Meeting - June 24, 2019

## Attendees

No quorum today...

* Jim Truher
* Dongbo Wang
* Joey Aiello

## Notes

* PSGet 3.0 RFC has been updated, need to review with Steve when he's back
* Closed PR #138 on the basis that it misunderstood the current process
* Cursorily reviewed and briefly discussed PR 202 for side-by-side `pwsh` support
    * Initial reaction is that this is explicitly a non-goal
    * We are primarily trying to follow a fix-forward approach to `pwsh`
    * Complexity of managing multiple namespaces in package managers is very high
        * Also discouraged by distros
* Process on aging PRs out?
    * `Waiting on Author` for some period of time?
    * Open question: do we give it a number and reject it, or do we just close the PR?
        * The former provides more visibility to others who may want to pick it up
        * PR after would move it to 2-Draft-Accepted with the existing number
    * Probably won't use automation to remove `Waiting on Author`
        * Only the author's response would change it
* Still haven't codified the way that experimental features reduce the burden to RFCs
* Jim: "code speaks louder than words"
    * Jim did ulimit/umask implementation before he submitted the PR
    * At a bare minimum, input/output examples are necessary
    * Dongbo: "if you're really going to implement it, you need to think about all the details up front"
    * We don't want to think of the implementation plan, but having a design is critical
    * One great example is Test-Connection changes (#172)
        * Only 121 lines, and it gives everything you need to know about input, output, parameter sets, etc.
* We may need to be more direct about possible breakage that RFCs cause
* Reviewed login shell RFC (PR #186)
    * Pending performance measurements of Rob's new method, we will take his method of `pwsh -l` / `pwsh -LoginShell` instead of using `pwsh-login`
    * Open question: on a container, does Docker implicitly start the pwsh entrypoint from sh?

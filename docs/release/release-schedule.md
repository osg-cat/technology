OSG Software Release Schedule
-----------------------------

To help with predictability and align with the GOC update schedule, we have adopted a fixed release schedule. OSG Software will be released on the second Tuesday of the month. Urgent releases could happen on the fourth Tuesday of the month.

Release Cycle Phases
--------------------

Various "Freeze" dates will happen in the weeks leading up to the release. They are as follows:

-   Development Freeze (end of the day on the Monday two weeks before release):
    -   The remaining testing is planned for this release.
    -   Packages are considered for inclusion in the release on this date.
    -   Some additional work on the packages may be done later in the day.
    -   All packages to be tested, need to be promoted to the testing repository by the end of the day.
    -   We will schedule testing for all packages at this date.
    -   If a package is promoted after this date, it may not get tested in time for the release.
-   Package Freeze (end of the day on the Monday one week before release):
    -   The intended release set is determined.
    -   Packages that have been tested will be promoted to the prerelease repository as they are ready.
    -   New packages that have short predictable testing cycle will be accepted for release.
-   Prerelease Freeze (2 PM Central on the Friday before release):
    -   Last minute decision of which packages are included.
    -   Packages will be run through the automated tests just prior to release.
    -   Tarball clients will be created just prior to release.
-   Release (second or fourth Tuesday of the month):
    -   The software packages are placed in the release repositories.
    -   The release is announced.

If a freeze date falls on a holiday, it will be moved to the next business day. Exceptions for freezes may be made at the discretion of the Release Manager.

Example Schedule
----------------

Example schedule for the September 2017 release:

| Phase | Date |
| :---- | :--- |
| Development Freeze | Monday, 8/28 at 5 PM | 
| Package Freeze | Monday, 9/5 at 5 PM | 
| Prerelease Freeze | Friday, 9/9 at 2 PM | 
| Release | Tuesday, 9/12 |


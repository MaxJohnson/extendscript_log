#### 1.4.0 (2021-05-27)

##### New Features

*  add option for custom log file output directory
*  add stack trace to error object

##### Other Changes
*  removed source from error object

#### 1.3.2 (2021-05-27)

##### Other Changes
*  Fixes for error object handling
*  Properly escape control characters for json
*  description and comments update
*  Rollback uppercase change cause it broke things and was not useful.
*  Bug fix: added quotes around key values in JSON string sent via CSXSEvent()
*  [Bug Fix]: don't stringify a string... added check to return original string if string passed to _stringifyMessage(). Was always wrapping strings in quotes...
*  Documentation and license typos
*  First push...
*  Initial commit
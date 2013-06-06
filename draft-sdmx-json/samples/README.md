## Sample Live Data Sets

Following live data flows are available for testing.

### Harmonised Index of Consumer Prices Sample

- Web Service: http://live-test-ws-7.nodejitsu.com
- Agency: ECB
- Data flow: ECB_ICP1
- Data flow version: 1.0
- Key dimensions:
	- Frequency (FREQ): M
	- Reference Area (REF_AREA): AT, BE, CY, DE, ES, FI, FR, GR, IE, IT, LU, MT, NL, PT, SI, SK, U2
	- Adjustment indicator (ADJUSTMENT): N
	- Classification - ICP context (ICP_ITEM): 000000, 010000, 011000, 011100, 011200, 011300, 011400, 011500, 011600, 011700, 011800, 011900, 012000, 012100, 012200, 020000, 021000, 021100, 021200, 021300, 022000, 030000, 031000, 031100, 031200, 031300, 031400, 032000, 040000, 041000, 043000, 043100, 043200, 044000, 044100, 044200, 044300, 044400, 045000, 045100, 045200, 045300, 045400, 045500, 050000, 051000, 051100, 051200, 051300, 052000, 053000, 0531_2, 053300, 054000, 055000, 056000, 056100, 056200, 060000, 061000, 061100, 0612_3, 062000, 0621_3, 062200, 063000, 070000, 071000, 071100, 0712_4, 072000, 072100, 072200, 072300, 072400, 073000, 073100, 073200, 073300, 073400, 073500, 073600, 080000, 081000, 082_30, 090000, 091000, 091100, 091200, 091300, 091400, 091500, 092000, 0921_2, 092300, 093000, 093100, 093200, 093300, 0934_5, 094000, 094100, 094200, 095000, 095100, 095200, 0953_4, 096000, 100000, 110000, 111000, 111100, 111200, 112000, 120000, 121000, 121100, 1212_3, 123000, 123100, 123200, 124000, 125000, 125200, 125300, 125400, 125500, 126000, 126200, 127000
	- Institution originating the data flow (STS_INSTITUTION): 4
	- Series variation - ICP context (ICP_SUFFIX): INX
- Time Range (TIME_PERIOD): 1996 - 2009
- Supported Parameters: startPeriod, endPeriod, dimensionAtObservation, detail
- Sample query: http://live-test-ws-7.nodejitsu.com/data/ECB_ICP1/M.PT+FI.N.073000.4.INX?startPeriod=2009&endPeriod=2009
- Please note that this server runs with very limited hardware resources. Large requests may result in the HTTP error 503 if
the server runs out of memory and crashes. Server will then restart automatically.
- Please use the following optimized request if you want to retrieve the complete dataset: http://live-test-ws-7.nodejitsu.com/data/ECB_ICP1/.....



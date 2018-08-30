Cálculo de Nota Fiscal de Venda

LParameters PLL_CondPag

LOCAL VLN_TotalDesc, VLN_DesAcrValue, VLN_TotalLiquido, VLN_DifDesAcr

store 0 to VLN_TotalDesc, VLN_DifDesAcr

select J11T

replace all j11t.j11_008_b with j11t.j11_008_b + abs(j11t.j11_033_b), ;
			j11t.j11_033_b with 0

sum (j11t.j11_008_b - j11t.j11_033_b) to VLN_TotalLiquido

scan
	with VGO_Gen
		.FOL_SetParameter(1, "J11")
		.FOL_SetParameter(2, j11t.ukey)
		.VOA_Index[1] = "J22_PAR + J22_UKEYP + J22_005_C"
		.VOA_Index[2] = "A40_UKEY"
		.VOA_Index[3] = "J22_PAR + J22_UKEYP"
		.FOL_EditCursor("J22_J11_FRM", "J22_FRM", "J22T", "Z", 3, 2)

		.FOL_SetParameter(1, "J11")
		.FOL_SetParameter(2, j11t.ukey)
		.VOA_Index[1] = "J15_PAR + J15_UKEYP"
		.FOL_EditCursor("J15_J11_FRM", "J15_J11_FRM", "J15T", "Z", 1, 2)
	endwith

	with this.OOL_J11_Functions
		if VGO_Gen.FOL_FindExpression("T06_UKEY", "T06_UKEY", j11t.t06_ukey)
			.VOC_T28CM  = t06_ukey.t28_ukeyd
			.VOC_T28Com = t06_ukey.t28_ukeyc
			.VOC_T28Fin = t06_ukey.t28_ukeyb
			.VOC_T28Con = t06_ukey.t28_ukeya
			.VOC_T28Doc = t06_ukey.t28_ukey
		endif

		.FOL_CalcDesAcr(VLN_TotalLiquido)

		*- Acumula desconto da nota
		VLN_TotalDesc = VLN_TotalDesc + j11t.j11_033_b

		.FOL_CalcImp()
		.FOL_CalcFormulas()
		.FOL_CalcComission()
	endwith
	VGO_Gen.FOL_CreateSqlString("J11T", "J11", EMPTY(J11T.status), .F., "QTDORI, D04_008_C, D04_001_C")

	VGO_Gen.FOL_SaveCursor(.NULL., "J22T", "J22_J10T", "UKEY")
	VGO_Gen.FOL_SaveCursor(.NULL., "J15T", "J15_J10T", "UKEY")

	this.OOL_RecalculateRateio.FOU_Execute("B04TTT", "J11", j11t.ukey, "J11_021_B", ;
		"B04TTT", j11t.j11_021_b, .F., "A11_001_C, A11_003_C, A56_001_C, A56_003_C, A11_005_N, " + ;
		"A11_008_N, A56_005_N, A56_008_N")
	select J11T
endscan

this.OOL_J11_Functions.FOL_CalcTotals()

go top in J11T

select J06T
go top in J06T

replace j10_034_b with VLN_TotalDesc in J10

*- Trazendo o valor financeiro
if seek(2, "J06T", "TAG1") && Total para integração do Financeiro
	VLN_Value = j06t.j06_001_b
endif

with this
	.VON_FinValue 	  = VLN_Value
	.VON_LastFinValue = 0
	.VOC_LastA13Ukey  = nvl(j10.a13_ukey, "")
	.VOD_LastEmi 	  = j10.j10_003_d
endwith 

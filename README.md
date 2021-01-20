 <!doctype html>
<html lang="en">
  <head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>Westports ETP Export Booking Excel to Coprar Converter</title>
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"
        integrity="sha256-4+XzXVhsDmqanXGHaHvgh1gMQKX40OUvDEBTu8JcmNs="
        crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.8.0/jszip.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.8.0/xlsx.js"></script>
  </head>
  <body>          
        <script>

          parseExcel = function(file) {
            var reader = new FileReader();

            reader.onload = function(e) {
              var data = e.target.result;
              var workbook = XLSX.read(data, {
                type: 'binary'
              });

              workbook.SheetNames.forEach(function(sheetName) {
                // Here is your object
                //var XL_row_object = XLSX.utils.sheet_to_row_object_array(workbook.Sheets[sheetName]);
                var XL_row_object = XLSX.utils.sheet_to_csv(workbook.Sheets[sheetName]);
                //var json_object = JSON.stringify(XL_row_object);
                //console.log(XL_row_object)
                //console.log(json_object);
                var line = 0;
                var contcount = 0;
                var allRows = XL_row_object.split(/\r?\n|\r/);
                var table = '<table>';
                var dt = new Date();
                var refno = get_date_str(dt, "");
                var edi = "UNB+UNOC:4+PSA+WPT+"+$("#recv_code").val()+"+"+get_date_str(dt, "daterawonly")+":"+get_date_str(dt, "timetominrawonly")+"+"+refno+"'\n";
                edi += "UNH+"+refno+"+COPRAR:D:00B:UN:SMDG21+LOADINGCOPRAR'\n";
                line++;
                
                //process header
                var report_dt = ""; var voyage = ""; var vslname = ""; var callsign = ""; var opr = "";
                for (var singleRow = 0; singleRow < allRows.length; singleRow++) {
                    if(singleRow>6) break;
                    let rowCells = allRows[singleRow].split(',');
                    if(singleRow==1) {
                        let report_date = new Date(rowCells[1]);
                        report_dt = get_date_str(report_date, "");
                    }
                    if(singleRow==3) {
                        if(typeof rowCells[3]!="undefined") {
                            let tmp = rowCells[3].split('/');
                            voyage = tmp[0];
                            callsign = tmp[1];
                            opr = tmp[2];
                            vslname = rowCells[1];
                        }
                    }
                }                
                edi += "BGM+45+"+report_dt+"+5'\n"; line++;
                edi += "TDT+20+"+voyage+"+1++172:ZZZ+++"+callsign+":103::"+vslname+"'\n"; line++;
                edi += "RFF+VON:"+voyage+"'\n"; line++;
                let tmp; let dim;
                for (singleRow = 0; singleRow < allRows.length; singleRow++) {
                  if(typeof allRows[singleRow]!="undefined") {
                    let rowCells = allRows[singleRow].split(',');
                    if(singleRow>7) {
                        contcount++;
                        if(typeof rowCells[1]!="undefined" && typeof rowCells[7]!="undefined") edi += "EQD+CN+"+rowCells[1]+"+"+rowCells[7]+":102:5+++5'\n"; line++;
                        if(typeof rowCells[6]!="undefined") edi += "LOC+11+"+rowCells[6]+":139:6'\n"; line++;
                        if(typeof rowCells[19]!="undefined") edi += "LOC+9+"+rowCells[19]+":139:6'\n"; line++;
                        if(typeof rowCells[13]!="undefined") edi += "MEA+AAE+VGM+KGM:"+rowCells[13]+"'\n"; line++;
                        if(typeof rowCells[8]!="undefined") edi += "FTX+AAI+++"+rowCells[8]+"'\n"; line++;
                        if(typeof rowCells[14]!="undefined" && rowCells[14]!="" && $.trim(rowCells[14])!="/") {
                          tmp = rowCells[14].split('/');
                          edi += "DGS+IMD+"+tmp[0]+"+"+tmp[1]+"'\n"; line++;
                        }
                        if(typeof rowCells[17]!="undefined" && $.trim(rowCells[17])!="" && $.trim(rowCells[17])!="/") {
                          tmp = rowCells[17].split(',');
                          for(let i=0; i<tmp.length; i++) {
                              dim = rowCells[17].split('/');
                              if($.trim(dim[0])=="OF") {
                                  edi += "DIM+5+CMT:"+$.trim(dim[1])+"'\n"; line++;
                              }
                              if($.trim(dim[0])=="OB") {
                                  edi += "DIM+6+CMT:"+$.trim(dim[1])+"'\n"; line++;
                              }
                              if($.trim(dim[0])=="OR") {
                                  edi += "DIM+7+CMT::"+$.trim(dim[1])+"'\n"; line++;
                              }
                              if($.trim(dim[0])=="OL") {
                                  edi += "DIM+8+CMT::"+$.trim(dim[1])+"'\n"; line++;
                              }
                              if($.trim(dim[0])=="OH") {
                                  edi += "DIM+9+CMT:::"+$.trim(dim[1])+"'\n"; line++;
                              }
                          }
                        }
                        if(typeof rowCells[25]!="undefined" && $.trim(rowCells[25])!="" && $.trim(rowCells[25])!="/") {
                          let tmp = rowCells[25].split(',');
                          if(tmp[0]=="L") {
                              edi += "SEL+"+tmp[1]+"+CA'\n"; line++; //seal L - CA, S - SH, M - CU
                          }
                          if(tmp[0]=="S") {
                              edi += "SEL+"+tmp[1]+"+SH'\n"; line++; //seal L - CA, S - SH, M - CU
                          }
                          if(tmp[0]=="M") {
                              edi += "SEL+"+tmp[1]+"+CU'\n"; line++; //seal L - CA, S - SH, M - CU
                          }
                        }
                        if(typeof rowCells[15]!="undefined" && $.trim(rowCells[15])!="" && $.trim(rowCells[15])!="/") {
                          edi += "TMP+2+"+rowCells[15]+":CEL'\n"; line++;
                        }
                        if(typeof rowCells[12]!="undefined" && $.trim(rowCells[12])!="" && $.trim(rowCells[12])!="/") {
                          edi += "FTX+AAA+++"+rowCells[12]+"'\n"; line++;
                        }
                        if(typeof rowCells[18]!="undefined" && $.trim(rowCells[18])!="" && $.trim(rowCells[18])!="/") {
                          edi += "FTX+HAN++"+rowCells[18]+"'\n"; line++;
                        }
                        if(typeof rowCells[18]!="undefined") edi += "NAD+CF+"+rowCells[18]+":160:ZZZ'\n"; line++;//box
                        edi += "NAD+CA+"+opr+":160:ZZZ'\n"; line++; //vsl
                        if(typeof rowCells[27]!="undefined") edi += "NAD+GF+"+rowCells[27]+":160:ZZZ'\n"; line++; //slot
                    }                    

                    /*if (singleRow === 0) {
                      table += '<thead>';
                      table += '<tr>';
                    } else {
                      table += '<tr>';
                    }
                    var rowCells = allRows[singleRow].split(',');
                    for (let rowCell = 0; rowCell < rowCells.length; rowCell++) {
                      if (singleRow === 0) {
                        table += '<th>';
                        table += rowCells[rowCell];
                        table += '</th>';
                      } else {
                        table += '<td>';
                        table += rowCells[rowCell];
                        table += '</td>';
                      }
                    }
                    if (singleRow === 0) {
                      table += '</tr>';
                      table += '</thead>';
                      table += '<tbody>';
                    } else {
                      table += '</tr>';
                    }*/
                  }
                } 
                edi += "CNT+16:"+contcount+"'\n"; line++; line++;
                edi += "UNT+"+line+"+"+refno+"'\n";
                edi += "UNZ+1+"+refno+"'";
                //table += '</tbody>';
                //table += '</table>';
                $('#my_file_output').val(edi);

              })

            };

            reader.onerror = function(ex) {
              console.log(ex);
            };

            reader.readAsBinaryString(file);
          };
        
        var oFileIn;

        $(function() {
            oFileIn = document.getElementById('my_file_input');
            if(oFileIn.addEventListener) {
                oFileIn.addEventListener('change', filePicked, false);
            }
        });


        function filePicked(oEvent) {
            // Get The File From The Input
            var oFile = oEvent.target.files[0];
            var sFilename = oFile.name;
            parseExcel(oFile)
            
        }
        
        function copy() {
            /* Get the text field */
            var copyText = document.getElementById("ediholder");

            /* Select the text field */
            copyText.select();
            copyText.setSelectionRange(0, 99999); /* For mobile devices */

            /* Copy the text inside the text field */
            document.execCommand("copy");

            /* Alert the copied text */
            alert("Text Copied!");
        } 
        
        function get_date_str(d, type) {
            var now = d;
            var dt = now.getDate();
            dt = (String(dt).length<2)? String("0") + String(dt) : dt;
            var hrs = now.getHours();
            hrs = (String(hrs).length<2)? String("0") + String(hrs) : hrs;
            var min = now.getMinutes();
            min = (String(min).length<2)? String("0") + String(min) : min;
            var sec = now.getSeconds();
            sec = (String(sec).length<2)? String("0") + String(sec) : sec;
            var mth = (now.getMonth() + 1);
            mth = (String(mth).length<2)? String("0") + String(mth) : mth;
            if(type=="daterawonly") {
                return now.getFullYear()+''+String(mth)+''+String(dt);
            } else if(type=="timetominrawonly"){
                return String(hrs)+''+String(min);
            } else {
                return now.getFullYear()+''+String(mth)+''+String(dt)+''+String(hrs)+''+String(min)+''+String(sec);
            }
            //return now.getHours()+':'+String(min)+':'+String(sec);
        }
</script>
<div class="container">
    <div class="card" style="">
        <div class="card-body">
            <h5 class="card-title">Westports ETP Export Booking Excel to Coprar Converter</h5>
            <div class="form-group">
                <label for="my_file_input">Export booking excel file:</label><input class="form-control" type="file" id="my_file_input" />
            </div>
            <div class="form-group">
                <label for="recv_code">Receiver Code:</label><input class="form-control" type="text" id="recv_code" value="RECEIVER" />
            </div>
            <div class="form-group"><textarea class="form-control" rows="20" cols="40" id='my_file_output'></textarea></div>
        </div>
    </div>
</div>
</body>
</html>

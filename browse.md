---
---
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<h1>Degree <span id="degree">(unknown)</span> extensions of &#8474;<sub id="prime">(unknown)</sub></h1>
<div id="info"></div>
<table id="table">
  <tr>
    <th>Polynomial</th>
    <th>Galois group</th>
  </tr>
</table>
<p id="loading">Loading...</p>
<script type="text/javascript">
  $('document').ready(function (){
    // Read a page's GET URL variables and return them as an associative array.
    function getUrlVars()
    {
        var vars = [], hash;
        var hashes = window.location.href.slice(window.location.href.indexOf('?') + 1).split('&');
        for(var i = 0; i < hashes.length; i++)
        {
            hash = hashes[i].split('=');
            vars.push(hash[0]);
            vars[hash[0]] = hash[1];
        }
        return vars;
    }

    // given coefficients, makes a polynomial in HTML
    function polynomialToHTML(coeffs) {
      let html = '';
      for (var i=coeffs.length-1; i>=0; i--) {
        let c = coeffs[i];
        if (c === 0) { 
          continue;
        }
        let xi = (i===0) ? '' : (i===1) ? 'x' : ('x<sup>'+i+'</sup>');
        if (html.length === 0) {
          html += '' + (c===1?'':c===-1?'-':c) + xi;
        } else {
          if (c > 0) {
            html += ' + ' + (c===1?xi?'':c:c) + xi;
          } else {
            html += ' - ' + (c===-1?xi?'':-c:-c) + xi;
          }
        }
      }
      if (html.length === 0) {
        html = '0';
      }
      return html;
    }

    // given the string contents of a CSV file, populates the table
    function CSVtoTable($table, data, deg) {
      // get the rows of the table
      let allLines = data.split(/\r\n|\n/);
      let headers = allLines[0].split(',');
      let rows = [];
      for (var i = 1; i < allLines.length; i++) {
        if (allLines[i].length === 0) { continue; }
        let x = allLines[i].split(',')
        let coeffs = [];
        for (var j = 0; j <= deg; j++) {
          coeffs.push(parseInt(x[j], 10));
        }
        let tnum = parseInt(x[deg+1], 10);
        rows.push([coeffs, tnum]);
      }
      rows.sort(function (a,b) { return a[1] - b[1]; });
      // make the table
      let numunknown = 0;
      for (var i = 0; i<rows.length; i++) {
        let row = rows[i];
        let coeffs = row[0];
        let tnum = row[1];
        if (tnum === 0) {
          $table.append($('<tr></tr>').append($('<td>'+polynomialToHTML(coeffs)+'</td>')).append($('<td>unknown</td>')));
          numunknown += 1;
        } else {
          $table.append($('<tr></tr>').append($('<td>'+polynomialToHTML(coeffs)+'</td>')).append($('<td style="white-space: nowrap;">'+deg+'T'+tnum+' (<a href="https://hobbes.la.asu.edu/Groups/group-data/d'+deg+'t'+tnum+'.html" target="_blank">JJ</a>, <a href="http://groupnames.org/#?'+deg+'T'+tnum+'" target="_blank">GN</a>)</td>')));
        }
      }
      if (numunknown > 0) {
        $('#info').append($('<p>Note: '+numunknown+' unknown Galois groups</p>'));
      }
    }

    function suffixToText(x) {
      if (x==='e2') {
        return 'All fields with ramification degree 2';
      } else if (x==='e14') {
        return 'All fields with ramification degree 14';
      } else if (x==='tr') {
        return 'All totally ramified fields';
      } else if (x==='e4a') {
        return 'All fields with ramification degree 4, singly ramified and elementary abelian over its maximal unramified subfield';
      } else if (x==='e8a') {
        return 'All fields with ramification degree 8, singly ramified and elementary abelian over its maximal unramified subfield';
      } else {
        return 'Some unknown restriction'
      }
    }

    let getvars = getUrlVars();
    let prime = parseInt(getvars.prime, 10);
    let degree = parseInt(getvars.degree, 10);
    let suffix = getvars.suffix;
    let csvfile = 'p'+prime+'_d'+degree+(suffix?'_'+suffix:'')+'.csv';
    let csvurl = 'https://raw.githubusercontent.com/cjdoris/pAdicGaloisGroupTables/master/' + csvfile;
    $('#prime').text(prime);
    $('#degree').text(degree);
    if (suffix) {
      $('#info').append('<p>Note: ' + suffixToText(suffix) + '</p>')
    }
    if (getvars.dupes) {
      $('#info').append($('<p>Note: Possibly several polynomials per field</p>'))
    }
    $('#info').append($('<p>File: <a href="'+csvurl+'">'+csvfile+'</a></p>'))
    if (degree <= 12) {
      $('#info').append($('<p>Also in: <a href="https://hobbes.la.asu.edu/LocalFields/basic-table.cgi?prime='+prime+'&degree='+degree+'" target="_blank">LFDB</a>, <a href="http://www.lmfdb.org/LocalNumberField/?p='+prime+'&n='+degree+'" target="_blank">LMFDB</a></p>'))
    }
    else if (degree <= 14) {
      $('#info').append($('<p>Also in: <a href="http://www.lmfdb.org/LocalNumberField/?p='+prime+'&n='+degree+'" target="_blank">LMFDB</a></p>'))
    }
    $.ajax({
      type: 'GET',
      url: csvurl,
      dataType: 'text',
      success: function (data) { $('#loading').remove(); CSVtoTable($('table'), data, degree); },
      error: function (xhr,st,err) { $('#loading').empty().text('Error loading CSV file! (check URL parameters)').css('color','red'); },
    });
  })
</script>

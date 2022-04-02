<script>

// https://stackoverflow.com/questions/1284314/easter-date-in-javascript
function get_easter_date(y) {

    var date, a, b, c, m, d;

    // Instantiate the date object.
    date = new Date;

    // Set the timestamp to midnight.
    date.setHours( 0, 0, 0, 0 );

    // Set the year.
    date.setFullYear( y );

    // Find the golden number.
    a = y % 19;

    // Choose which version of the algorithm to use based on the given year.
    b = ( 2200 <= y && y <= 2299 ) ?
        ( ( 11 * a ) + 4 ) % 30 :
        ( ( 11 * a ) + 5 ) % 30;

    // Determine whether or not to compensate for the previous step.
    c = ( ( b === 0 ) || ( b === 1 && a > 10 ) ) ?
        ( b + 1 ) :
        b;

    // Use c first to find the month: April or March.
    m = ( 1 <= c && c <= 19 ) ? 3 : 2;

    // Then use c to find the full moon after the northward equinox.
    d = ( 50 - c ) % 31;

    // Mark the date of that full moon—the "Paschal" full moon.
    date.setMonth( m, d );

    // Count forward the number of days until the following Sunday (Easter).
    date.setMonth( m, d + ( 7 - date.getDay() ) );

    // Gregorian Western Easter Sunday
    return date;
}

export default {
    data() {

        var special_dates = [
            {"easter_diff": 46, "verdict": 3, "desc": "Środa Popielcowa"},
            {"easter_diff": 2, "verdict": 3, "desc": "Wielki Piątek"},
            {"easter_diff": 1, "verdict": 2, "desc": "Wielka Sobota"},
            {"easter_diff": 0, "verdict": 0, "desc": "Wielkanoc"},
            {"easter_diff": -5, "verdict": 0, "desc": "piątek Oktawy Wielkanocnej"},
            {"easter_diff": -42, "verdict": 0, "desc": "Uroczystość Wniebowstąpienia Pańskiego"},
            {"easter_diff": -56, "verdict": 0, "desc": "Uroczystość Najświętszej Trójcy"},
            {"easter_diff": -60, "verdict": 0, "desc": "Uroczystość Najświętszego Ciała i Krwi Chrystusa", "aka": "Boże Ciało"},
            {"easter_diff": -68, "verdict": 0, "desc": "Uroczystość Najświętszego Serca Pana Jezusa"},
            // ---
            {"date": [1, 1], "verdict": 0, "desc": "Uroczystość Świętej Bożej Rodzicielki Maryi", "aka": "Nowy Rok"},
            {"date": [1, 6], "verdict": 0, "desc": "Uroczystość Objawienia Pańskiego", "aka": "Święto Trzech Króli"},
            {"date": [3, 19], "verdict": 0, "desc": "Uroczystość świętego Józefa"},
            {"date": [3, 25], "verdict": 0, "desc": "Uroczystość Zwiastowania Pańskiego"},
            {"date": [4, 23], "verdict": 0, "desc": "Uroczystość świętego Wojciecha patrona Polski"},
            {"date": [5, 3], "verdict": 0, "desc": "Uroczystość Najświętszej Maryi Panny Królowej Polski"},
            {"date": [5, 8], "verdict": 0, "desc": "Uroczystość świętego Stanisława ze Szczepanowa"},
            {"date": [6, 24], "verdict": 0, "desc": "Uroczystość narodzenia świętego Jana Chrzciciela"},
            {"date": [6, 29], "verdict": 0, "desc": "Uroczystość świętych apostołów Piotra i Pawła"},
            {"date": [8, 15], "verdict": 0, "desc": "Uroczystość wniebowzięcia Najświętszej Maryi Panny", "aka": "Matki Boskiej Zielnej"},
            {"date": [8, 26], "verdict": 0, "desc": "Uroczystość Najświętszej Maryi Panny Częstochowskiej"},
            {"date": [10, 20], "verdict": 0, "desc": "Uroczystość świętego Jana Kantego, patrona Polski i Litwy"},
            {"date": [11, 1], "verdict": 0, "desc": "Uroczystość Wszystkich Świętych"},
            // TODO: last Sunday before the advent
            // {"date": [??], "verdict": 0, "desc": "Uroczystość Jezusa Chrystusa Króla Wszechświata"},
            {"date": [12, 8], "verdict": 0, "desc": "Uroczystość Niepokalanego Poczęcia Najświętszej Maryi Panny"},
            {"date": [12, 24], "verdict": 1, "desc": "Wigilia Narodzenia Pańskiego"},
            {"date": [12, 25], "verdict": 0, "desc": "Uroczystość Narodzenia Pańskiego", "aka": "Boże Narodzenie"},
            ];

        var today = new Date();
        today.setHours(0, 0, 0, 0);
        var easter = get_easter_date(today.getFullYear());

        console.log("Today's date: " + today);
        console.log("Easter date:  " + easter);

        var easter_diff = Math.round((easter.getTime() - today.getTime()) / (24 * 60 * 60 * 1000));

        console.log("Days until Easter:  " + easter_diff);

        var verdict = null;
        var date_desc = null;
        var date_desc_aka = null;

        for (const entry of special_dates) {
            if ("easter_diff" in entry) {
                if (easter_diff != entry["easter_diff"])
                    continue;
            } else {
                if (today.getMonth() + 1 != entry["date"][0] || today.getDate() != entry["date"][1]) {
                    continue;
                }
            }
            verdict = entry["verdict"];
            date_desc = entry["desc"];
            date_desc_aka = entry["aka"];
            break;
        }

        if (verdict == null) {
            if (today.getDay() == 5) {
                verdict = 2;
                date_desc = "piątek";
            } else {
                verdict = 0;
            }
        }

        var explanation = null;

        if (date_desc != null) {
            explanation = "Jest " + date_desc;
        }

        if (date_desc_aka != null) {
            explanation += " (czyli " + date_desc_aka + ")";
        }

        switch (verdict) {
            case 1:
                explanation += ".  Można zachować post ze względu na tradycję narodową";
                break;
            case 2:
                explanation += ", więc należy zachować wstrzemięźliwość od pokarmów mięsnych";
                break;
            case 3:
                explanation += ", więc obowiązuje post ścisły";
                break;
        }

        if (date_desc != null) {
            explanation += ".";
        }

        return {
            verdict: verdict >= 2,
            explanation: explanation,
            show_explanation: false,
        }
    }
}
</script>

<template>
  <v-app>
    <v-main>
      <v-container fluid>
        <v-row class="text-center">
          <v-col cols="12">
            <h1 class="font-weight-light" style="font-size: 5vw">Czy dzisiaj jest post?</h1>
            <p style="font-size: 20vw">{{ verdict ? "Tak" : "Nie"}}</p>
            <p v-if="explanation" class="text-grey font-weight-light" @click="show_explanation = !show_explanation" style="font-size: 3vw">
                <a v-if="!show_explanation">Jak to?</a>
                <a v-else>{{explanation}}</a>
            </p>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

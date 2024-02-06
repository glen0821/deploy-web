<!-- CertificateContent.svelte -->
<script>
  export let documentTitle = "";
  export let mywindow = "";
  export let completeName = "";
  export let address = "";
  export let age = "";
  export let birthDate = "";
  export let dateOfAppointment = "";
  export let purpose = "";
  export let gender = "";
  export let civilStatus = "";

  import nicolasLogo from "./images/nicolas.jpeg";
  import bacoorLogo from "./images/bacoor.jpeg";
  import clerk from "./images/clerk.png";
  import notedby from "./images/notedBy.png";
  import footer from "./images/indigencyFooter.png";

  const handleEvent = () => {
    mywindow.print();
    // mywindow.close();
  };
  setTimeout(handleEvent, 1000);

  function convertDateFormat(inputDate) {
    // Parse the input date string
    const parts = inputDate.split("-");
    const year = parseInt(parts[0]);
    const month = parseInt(parts[1]);
    const day = parseInt(parts[2]);

    // Create a Date object with the parsed values
    const dateObject = new Date(year, month - 1, day);

    // Get month name
    const monthNames = [
      "January",
      "February",
      "March",
      "April",
      "May",
      "June",
      "July",
      "August",
      "September",
      "October",
      "November",
      "December",
    ];
    const monthName = monthNames[dateObject.getMonth()];

    // Format the output date string
    // const outputDate = `${monthName} ${day}, ${year}`;

    return [monthName, day, year];
  }

  function getOrdinalSuffix(number) {
    // Check for special cases: 11th, 12th, 13th
    if (number % 100 >= 11 && number % 100 <= 13) {
      return "th";
    }

    // Regular cases
    switch (number % 10) {
      case 1:
        return "st";
      case 2:
        return "nd";
      case 3:
        return "rd";
      default:
        return "th";
    }
  }
</script>

<div class="box">
  <style>
    .box {
      width: 630px;
      height: 891px;
    }

    .box .group {
      position: fixed;
      width: 666px;
      height: 891px;
      top: 0;
      left: 0;
    }

    .box .overlap {
      position: relative;
      width: 630px;
      height: 891px;
      background-color: #ffffff;
    }

    .box .overlap-group {
      position: absolute;
      width: 222px;
      height: 52px;
      top: 489px;
      left: 214px;
    }

    .box .screenshot {
      position: absolute;
      width: 120px;
      height: 52px;
      top: 0;
      left: 102px;
      object-fit: cover;
    }

    .box .text-wrapper {
      position: absolute;
      width: 203px;
      top: 17px;
      left: 0;
      font-family: "Inter-Medium", Helvetica;
      font-weight: 500;
      color: #000000;
      font-size: 14px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .div {
      position: absolute;
      width: 172px;
      height: 366px;
      top: 210px;
      left: 37px;
    }

    .box .overlap-group-wrapper {
      position: absolute;
      width: 113px;
      height: 24px;
      top: 277px;
      left: 31px;
    }

    .box .overlap-group-2 {
      position: relative;
      width: 109px;
      height: 24px;
    }

    .box .text-wrapper-2 {
      width: 108px;
      left: 1px;
      position: absolute;
      top: 0;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #000000;
      font-size: 12px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .text-wrapper-3 {
      top: 14px;
      left: 0;
      position: absolute;
      width: 108px;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #0061bb;
      font-size: 8px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .overlap-wrapper {
      position: absolute;
      width: 172px;
      height: 366px;
      top: 0;
      left: 0;
    }

    .box .overlap-2 {
      position: relative;
      height: 366px;
    }

    .box .rectangle {
      top: 3px;
      left: 0;
      background-color: #727272;
      position: absolute;
      width: 170px;
      height: 363px;
      border: 1px solid;
      border-color: #000000;
    }

    .box .rectangle-2 {
      top: 0;
      left: 2px;
      background-color: #ffffff;
      position: absolute;
      width: 170px;
      height: 363px;
      border: 1px solid;
      border-color: #000000;
    }

    .box .div-wrapper {
      position: absolute;
      width: 113px;
      height: 24px;
      top: 16px;
      left: 30px;
    }

    .box .group-2 {
      position: absolute;
      width: 113px;
      height: 24px;
      top: 66px;
      left: 29px;
    }

    .box .group-3 {
      position: absolute;
      width: 113px;
      height: 24px;
      top: 331px;
      left: 30px;
    }

    .box .group-4 {
      position: absolute;
      width: 154px;
      height: 25px;
      top: 97px;
      left: 12px;
    }

    .box .text-wrapper-4 {
      width: 150px;
      left: 0;
      position: absolute;
      top: 0;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #000000;
      font-size: 12px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .text-wrapper-5 {
      top: 15px;
      left: 18px;
      position: absolute;
      width: 108px;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #0061bb;
      font-size: 8px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .group-5 {
      position: absolute;
      width: 113px;
      height: 24px;
      top: 157px;
      left: 30px;
    }

    .box .group-6 {
      position: absolute;
      width: 137px;
      height: 25px;
      top: 127px;
      left: 20px;
    }

    .box .text-wrapper-6 {
      width: 133px;
      left: 0;
      position: absolute;
      top: 0;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #000000;
      font-size: 12px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .text-wrapper-7 {
      top: 15px;
      left: 8px;
      position: absolute;
      width: 108px;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #0061bb;
      font-size: 8px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .group-7 {
      position: absolute;
      width: 113px;
      height: 24px;
      top: 186px;
      left: 30px;
    }

    .box .group-8 {
      position: absolute;
      width: 144px;
      height: 25px;
      top: 217px;
      left: 17px;
    }

    .box .text-wrapper-8 {
      position: absolute;
      width: 140px;
      top: 0;
      left: 0;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #000000;
      font-size: 12px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .text-wrapper-9 {
      top: 15px;
      left: 14px;
      position: absolute;
      width: 108px;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #0061bb;
      font-size: 8px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .group-9 {
      position: absolute;
      width: 136px;
      height: 25px;
      top: 303px;
      left: 21px;
    }

    .box .text-wrapper-10 {
      position: absolute;
      width: 132px;
      top: 0;
      left: 0;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #000000;
      font-size: 12px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .group-10 {
      position: absolute;
      width: 131px;
      height: 25px;
      top: 247px;
      left: 21px;
    }

    .box .text-wrapper-11 {
      width: 127px;
      left: 0;
      position: absolute;
      top: 0;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #000000;
      font-size: 12px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .republic-of-the {
      position: absolute;
      width: 316px;
      top: 52px;
      left: 157px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 12px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .text-wrapper-12 {
      position: absolute;
      width: 316px;
      top: 110px;
      left: 157px;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #000000;
      font-size: 20px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .BARANGAY {
      position: absolute;
      width: 316px;
      top: 168px;
      left: 157px;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #000000;
      font-size: 20px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .line {
      height: 3px;
      top: 146px;
      position: absolute;
      width: 555px;
      left: 37px;
    }

    .box .img {
      height: 1px;
      top: 151px;
      object-fit: cover;
      position: absolute;
      width: 555px;
      left: 37px;
    }

    .box .nicolas {
      position: absolute;
      width: 60px;
      height: 60px;
      top: 37px;
      left: 74px;
      object-fit: cover;
    }

    .box .photo {
      position: absolute;
      width: 82px;
      height: 80px;
      top: 176px;
      left: 505px;
    }

    .box .overlap-3 {
      position: relative;
      width: 80px;
      height: 80px;
      background-color: #ffffff;
      border: 1px solid;
      border-color: #000000;
      margin-top: 20px;
    }

    .box .text-wrapper-13 {
      position: absolute;
      width: 43px;
      height: 17px;
      top: 30px;
      left: 17px;
      font-family: "Inter-Medium", Helvetica;
      font-weight: 500;
      color: #000000;
      font-size: 11px;
      text-align: center;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .bacoor {
      position: absolute;
      width: 60px;
      height: 60px;
      top: 37px;
      left: 496px;
      object-fit: cover;
    }

    .box .notedby {
      position: absolute;
      width: 260px;
      height: 124px;
      top: 546px;
      left: 214px;
      object-fit: cover;
    }

    .box .footer {
      position: absolute;
      width: 629px;
      height: 168px;
      top: 723px;
      left: 0;
      object-fit: cover;
    }

    .box .overlap-4 {
      position: absolute;
      width: 191px;
      height: 36px;
      top: 240px;
      left: 214px;
    }

    .box .p {
      position: absolute;
      width: 191px;
      top: 0;
      left: 0;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      color: #000000;
      font-size: 13px;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .text-wrapper-14 {
      position: absolute;
      width: 186px;
      top: 21px;
      left: 0;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .complete-name {
      position: absolute;
      width: 186px;
      top: 288px;
      left: 214px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .birth-day {
      position: absolute;
      width: 95px;
      top: 289px;
      left: 365px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .address {
      width: 246px;
      left: 277px;
      position: absolute;
      top: 321px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .overlap-5 {
      position: absolute;
      width: 318px;
      height: 13px;
      top: 417px;
      left: 214px;
    }

    .box .purpose {
      width: 246px;
      top: 0;
      left: 72px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      position: absolute;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .text-wrapper-15 {
      width: 86px;
      top: 1px;
      left: 0;
      font-family: "Inter-Bold", Helvetica;
      font-weight: 700;
      position: absolute;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .appointment-date {
      position: absolute;
      width: 246px;
      top: 449px;
      left: 277px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .text-wrapper-16 {
      position: absolute;
      width: 37px;
      top: 289px;
      left: 334px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .text-wrapper-17 {
      width: 63px;
      left: 214px;
      position: absolute;
      top: 321px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .text-wrapper-18 {
      position: absolute;
      width: 286px;
      top: 391px;
      left: 214px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .text-wrapper-19 {
      position: absolute;
      width: 51px;
      top: 450px;
      left: 214px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }

    .box .text-wrapper-20 {
      position: absolute;
      width: 378px;
      top: 326px;
      left: 214px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
    }

    .box .text-wrapper-21 {
      position: absolute;
      width: 280px;
      top: 295px;
      left: 214px;
      font-family: "Inter-Regular", Helvetica;
      font-weight: 400;
      color: #000000;
      font-size: 11px;
      letter-spacing: 0;
      line-height: normal;
      white-space: nowrap;
    }
  </style>
  <div class="group">
    <div class="overlap">
      <div class="overlap-group">
        <img class="screenshot" src={clerk} />
        <div class="text-wrapper">Prepared By:</div>
      </div>
      <div class="div">
        <div class="overlap-group-wrapper">
          <div class="overlap-group-2">
            <div class="text-wrapper-2">Alfredo F.&nbsp;&nbsp;Arevalo</div>
            <div class="text-wrapper-3">Barangay Chairman</div>
          </div>
        </div>
        <div class="overlap-wrapper">
          <div class="overlap-2">
            <div class="rectangle"></div>
            <div class="rectangle-2"></div>
            <div class="div-wrapper">
              <div class="overlap-group-2">
                <div class="text-wrapper-2">Alfredo F.&nbsp;&nbsp;Arevalo</div>
                <div class="text-wrapper-3">Barangay Chairman</div>
              </div>
            </div>
            <div class="group-2">
              <div class="overlap-group-2">
                <div class="text-wrapper-2">Manuel F. Arevalo</div>
                <div class="text-wrapper-3">Councilor</div>
              </div>
            </div>
            <div class="group-3">
              <div class="overlap-group-2">
                <div class="text-wrapper-2">Lyziel S. Gonzales</div>
                <div class="text-wrapper-3">Barangay Treasurer</div>
              </div>
            </div>
            <div class="group-4">
              <div class="text-wrapper-4">Domingo D. Jimenez Jr.</div>
              <div class="text-wrapper-5">Councilor</div>
            </div>
            <div class="group-5">
              <div class="overlap-group-2">
                <div class="text-wrapper-2">Leona V. Frias</div>
                <div class="text-wrapper-3">Councilor</div>
              </div>
            </div>
            <div class="group-6">
              <div class="text-wrapper-6">George L. Gawaran Jr</div>
              <div class="text-wrapper-7">Councilor</div>
            </div>
            <div class="group-7">
              <div class="overlap-group-2">
                <div class="text-wrapper-2">Juddy E. Better</div>
                <div class="text-wrapper-3">Councilor</div>
              </div>
            </div>
            <div class="group-8">
              <div class="text-wrapper-8">Jeric C. Delos Santos</div>
              <div class="text-wrapper-9">Councilor</div>
            </div>
            <div class="overlap-group-wrapper">
              <div class="overlap-group-2">
                <div class="text-wrapper-2">Niña L. Cañete</div>
                <div class="text-wrapper-3">S.K. Chairman</div>
              </div>
            </div>
            <div class="group-9">
              <div class="text-wrapper-10">Ricelyn A. Custodio</div>
              <div class="text-wrapper-7">Barangay Secretary</div>
            </div>
            <div class="group-10">
              <div class="text-wrapper-11">Nelson S. Campasivo</div>
              <div class="text-wrapper-7">Barangay Chairman</div>
            </div>
          </div>
        </div>
      </div>
      <p class="republic-of-the">
        Republic of the Philippines<br />Province of Cavite<br />City of BACOOR
      </p>
      <div class="text-wrapper-12">BARANGAY SAN NICOLAS III</div>
      <div class="BARANGAY">
        BARANGAY CERTIFICATION&nbsp;&nbsp;&nbsp;&nbsp;OF INDIGENCY
      </div>
      <svg
        class="line"
        width="555"
        height="3"
        viewBox="0 0 555 3"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <line y1="1.5" x2="555.001" y2="1.5" stroke="black" stroke-width="3" />
      </svg>
      <div style="height: 5px;"></div>
      <svg
        class="img"
        width="555"
        height="1"
        viewBox="0 0 555 1"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <line y1="0.5" x2="555.001" y2="0.5" stroke="black" />
      </svg>
      <img class="nicolas" src={nicolasLogo} />
      <div class="photo">
        <div class="overlap-3"><div class="text-wrapper-13">Photo</div></div>
      </div>
      <img class="bacoor" src={bacoorLogo} />
      <img class="notedby" src={notedby} />
      <img class="footer" src={footer} />
      <div class="overlap-4">
        <p class="p">TO WHOM IT MAY CONCERN:</p>
        <p class="text-wrapper-14">This is to certify that</p>
      </div>
      <div class="complete-name" style="text-decoration: underline;">
        {completeName}
      </div>
      <div class="birth-day" style="text-decoration: underline;">
        {birthDate}
      </div>
      <div class="address" style="text-decoration: underline;">
        {address}
      </div>
      <div class="overlap-5">
        <div class="purpose" style="text-decoration: underline;">
          {purpose}
        </div>
        <div class="text-wrapper-15">Purpose:</div>
      </div>
      <div class="appointment-date" style="text-decoration: underline;">
        {dateOfAppointment}
      </div>
      <div class="text-wrapper-16">born</div>
      <div class="text-wrapper-17">address at</div>
      <p class="text-wrapper-18">
        This Certification is issued upon his/her request for:
      </p>
      <div class="text-wrapper-19">Issued:</div>
      <p class="text-wrapper-20">
        San Nicolas III Bacoor City, Cavity is known to be of good moral
        character and law abiding citizen in the community, furthermore, that
        the above mention name is INDIGENT and has no fix source of income.
      </p>
      <p class="text-wrapper-21">
        is a bonafide resident/voter with postal and residence
      </p>
    </div>
  </div>
</div>

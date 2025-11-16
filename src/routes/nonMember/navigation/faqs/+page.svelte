<script>
  import Navbar from "../../../components/Navbar.svelte";
</script>

<head>
  <title>Shining Light Baptist Church</title>
</head>

<div class="bg-gray-200 min-h-screen w-full overflow-auto">
  <!-- Navigation Bar -->
  <Navbar />
  
  <section class="relative mt-0 h-[700px] overflow-hidden bg-white">
    <!-- Video Background -->
    <video autoplay muted loop playsinline class="absolute inset-0 w-full h-full object-cover z-0">
      <source src="/faqsbg.mp4" type="video/mp4" />
      Your browser does not support the video tag.
    </video>

    <!-- Fully Transparent FAQ Section -->
    <div class="w-full px-4 py-12 relative z-10 h-[700px] flex justify-center items-start">
      <div class="w-full max-w-4xl p-6 md:p-10 rounded-xl overflow-y-auto h-full bg-white bg-opacity-50">
        <h2 class="text-3xl sm:text-4xl md:text-5xl font-bold mb-8 text-center text-[#003366]">FAQs</h2>

        <div class="space-y-4">

          <!-- FAQ Item -->
          <details class="p-4 rounded-lg border border-gray-300">
            <summary class="cursor-pointer text-base sm:text-lg font-semibold text-[#003366]">How Shining Light established?</summary>
            <p class="mt-2 text-sm sm:text-base text-gray">
              Shining Light Baptist Church started as a Bible study with Pastor Roger in May 2006.
              In January 2008, it became a mission church, and by January 2014, it was officially organized
              as a church. The church was founded by Pastor Michael Cal in Mayapa, Calamba. Over the
              years, the church has grown and expanded, establishing four branches across Laguna,
              specifically in Calamba, Mauro, San Pedro, and Calauan.
            </p>
          </details>

          <details class="p-4 rounded-lg border border-gray-300">
            <summary class="cursor-pointer text-base sm:text-lg font-semibold text-[#003366]">Can I attend even if I'm not a member?</summary>
            <p class="mt-2 text-sm sm:text-base text-gray">
              Absolutely! Everyone is welcome to attend and worship with us.
            </p>
          </details>

          <details class="p-4 rounded-lg border border-gray-300">
            <summary class="cursor-pointer text-base sm:text-lg font-semibold text-[#003366]">Do you offer programs for children?</summary>
            <p class="mt-2 text-sm sm:text-base text-gray">
              Yes! We have.
            </p>
          </details>

          <details class="p-4 rounded-lg border border-gray-300">
            <summary class="cursor-pointer text-base sm:text-lg font-semibold text-[#003366]">How can I give or donate?</summary>
            <p class="mt-2 text-sm sm:text-base text-gray">
              You may visit our <a href="/nonMember/navigation/donate" class="text-blue-400 underline">donation page</a> or scan the GCash QR code to support the ministry.
            </p>
          </details>

          <details class="p-4 rounded-lg border border-gray-300">
            <summary class="cursor-pointer text-base sm:text-lg font-semibold text-[#003366]">Who is the Founder of Shining Light Baptist Church?</summary>
            <p class="mt-2 text-sm sm:text-base text-gray">
              Pastor Michael Cal
            </p>
          </details>

          <!-- Branch Schedules -->
          <details class="p-4 rounded-lg border border-gray-300">
            <summary class="cursor-pointer text-base sm:text-lg font-semibold text-[#003366]">Time of church every Sunday in every branch?</summary>
            <div class="mt-2 space-y-4 text-sm sm:text-base text-gray">
              <details class="p-4 rounded-lg border border-gray-100">
                <summary class="cursor-pointer font-semibold text-[#003366]">San Pedro</summary>
                <ul class="list-disc pl-5 mt-1">
                  <li>Sunday School (Children): 7:00 AM</li>
                  <li>Worship Service: 9:30 AM</li>
                  <li>Gospel Hour: 2:00 PM</li>
                </ul>
              </details>

              <details class="p-4 rounded-lg border border-gray-200">
                <summary class="cursor-pointer font-semibold text-[#003366]">Calamba</summary>
                <ul class="list-disc pl-5 mt-1">
                  <li>Sunday School (Children): 8:00 AM</li>
                  <li>Worship Service: 9:00 AM</li>
                </ul>
              </details>

              <details class="p-4 rounded-lg border border-gray-200">
                <summary class="cursor-pointer font-semibold text-[#003366]">Calauan</summary>
                <ul class="list-disc pl-5 mt-1">
                  <li>Worship Service: 9:00 AM</li>
                  <li>Sunday School (Children): 2:00 PM</li>
                </ul>
              </details>

              <details class="p-4 rounded-lg border border-gray-200">
                <summary class="cursor-pointer font-semibold text-[#003366]">Mauro</summary>
                <ul class="list-disc pl-5 mt-1">
                  <li>Worship Service: 10:00 AM</li>
                  <li>Sunday School (Children): 2:00 PM</li>
                </ul>
              </details>
            </div>
          </details>

          <!-- Church Activities -->
          <details class="p-4 rounded-lg border border-gray-300">
            <summary class="cursor-pointer text-base sm:text-lg font-semibold text-[#003366]">What are the other activities of the church?</summary>
            <div class="mt-2 space-y-4 text-sm sm:text-base text-gray">
              <details class="p-4 rounded-lg border border-gray-200">
                <summary class="cursor-pointer font-semibold text-[#003366]">Soul Winning (S.W.A.B)</summary>
                <p class="mt-1">An outreach program focused on sharing the gospel through personal evangelism in communities.</p>
              </details>

              <details class="p-4 rounded-lg border border-gray-200">
                <summary class="cursor-pointer font-semibold text-[#003366]">Distribution of Tracks</summary>
                <p class="mt-1">We hand out gospel tracks in public places to spread the Word of God and invite people to church.</p>
              </details>

              <details class="p-4 rounded-lg border border-gray-200">
                <summary class="cursor-pointer font-semibold text-[#003366]">Bible Study</summary>
                <p class="mt-1">Regular group Bible studies held in various locations to grow in faith and fellowship.</p>
              </details>

              <details class="p-4 rounded-lg border border-gray-200">
                <summary class="cursor-pointer font-semibold text-[#003366]">Seek Ye First</summary>
                <p class="mt-1">A midweek devotion and worship service focused on prioritizing God in daily life.</p>
              </details>
            </div>
          </details>

        </div>
      </div>
    </div>
  </section>

  <!-- Contact and Social Media -->
  <section class="flex flex-col justify-between overflow-auto bg-gray-100 p-6 text-center md:flex-row md:text-left border-t-4 border-b-4 border-black">
    <!-- Left Side -->
    <div class="mb-4 md:mb-0 md:w-1/3">
      <h3 class="text-lg font-bold text-[#003366]">Socmed Accounts (e.g., FB)</h3>
      <div class="mt-8 flex justify-center space-x-8 md:justify-start">
        <!-- Facebook - Calamba Branch -->
        <div class="flex flex-col items-center">
          <a href="https://www.facebook.com/ShiningLightCalamba" target="_blank">
            <img src="https://upload.wikimedia.org/wikipedia/commons/5/51/Facebook_f_logo_%282019%29.svg" 
                 class="h-10 w-10 rounded-full object-cover cursor-pointer" 
                 alt="Facebook Logo">
          </a>
          <p class="text-sm mt-1 font-semibold text-black">Calamba Branch</p>
        </div>
        <!-- Facebook - San Pedro Branch -->
        <div class="flex flex-col items-center">
          <a href="https://www.facebook.com/SLBCSanPedroLaguna" target="_blank">
            <img src="https://upload.wikimedia.org/wikipedia/commons/5/51/Facebook_f_logo_%282019%29.svg" 
                 class="h-10 w-10 rounded-full object-cover cursor-pointer" 
                 alt="Facebook Logo">
          </a>
          <p class="text-sm mt-1 font-semibold text-black">San Pedro Branch</p>
        </div>
      </div>
    </div>

    <!-- Middle Section -->
    <div class="flex flex-col justify-between gap-8 md:w-1/3">
      <div>
        <h3 class="text-lg font-bold text-[#003366]">Contact us:</h3>
        <p><strong>San Pedro:</strong> (0921) 372 5922</p>
        <p><strong>Calamba:</strong> (0995) 460 2711</p>
        <p><strong>Calauan:</strong> (0935) 566 8843 | (0994) 722 9795</p>
        <p><strong>Mauro:</strong> (0995) 380 5525</p>
      </div>
    </div>

    <!-- Right Side (Email Section) -->
    <div class="flex flex-col justify-between md:w-1/3">
      <h3 class="text-lg font-bold text-[#003366]">Email Address:</h3>
      <p><strong>San Pedro:</strong> slbc_304@yahoo.com </p>
      <p><strong>Calamba:</strong> calamba.shininglight@gmail.com</p>
      <p><strong>Calauan:</strong> velascoarnel201@gmail.com</p>
      <p><strong>Mauro:</strong> inesjeffrey3@gmail.com</p>
    </div>
  </section>

  <footer class="overflow-auto bg-[#003366] p-4 text-center text-white">
    © Shining Light Baptist Church (Est. 2014)
  </footer>
</div>

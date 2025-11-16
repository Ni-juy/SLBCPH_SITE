<script>
    import Navbar from "../../../components/Navbar.svelte";
</script>

<head>
    <title>Shining Light Baptist Church</title>
  </head>
  

<div class="min-h-screen w-full overflow-auto bg-gray-200">
	<!-- Navigation Bar -->
	<Navbar />

    
    
    <section class="relative h-[700px] w-full flex flex-col items-center justify-center overflow-hidden">
        <!-- Background Video -->
        <div class="absolute inset-0 w-full h-full">
            <video autoplay loop muted playsinline class="w-full h-full object-cover">
                <source src="/bgvideo2.mp4" type="video/mp4" />
                Your browser does not support the video tag.
            </video>
        </div>
    
        <!-- Text Content -->
        <div class="absolute inset-0 text-white p-8 flex flex-col justify-center items-center z-10 bg-black/40">
            <h2 class="text-3xl md:text-4xl font-bold mb-4 text-center">Support Our Ministry</h2>
            <p class="text-base md:text-lg mb-6 text-center">
                Your generosity helps us continue spreading the Word and serving the community. Through your pledges, we are able to reach more lives and bring light where it’s most needed.
            </p>
            <a href="/nonMember/navigation/donate" class="bg-[#CC0000] text-white px-6 py-3 rounded-lg text-sm md:text-base hover:brightness-110 transition w-fit mx-auto">
                Donate Now
            </a>
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
  
  
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard Siswa Terintegrasi</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    
    <style>
        body { font-family: 'Inter', sans-serif; }
        .sidebar-transition { transition: transform 0.3s ease-in-out; }
        .custom-scrollbar::-webkit-scrollbar { width: 5px; height: 5px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: #1e293b; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #475569; border-radius: 5px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #64748b; }
        
        /* Playlist Scrollbar */
        .playlist-scroll::-webkit-scrollbar { width: 6px; }
        .playlist-scroll::-webkit-scrollbar-track { background: #f1f5f9; }
        .playlist-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 3px; }
        .playlist-scroll::-webkit-scrollbar-thumb:hover { background: #94a3b8; }

        .iframe-container { position: relative; width: 100%; height: 100%; overflow: hidden; background-color: white; border-radius: 0; }
        iframe { width: 100%; height: 100%; border: none; }
        .nav-item.active { background-color: #2563eb; color: white; }
        .nav-item.active i { color: white; }
        .loader { border: 4px solid #f3f3f3; border-top: 4px solid #3498db; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); display: none; z-index: 10; }
        @keyframes spin { 0% { transform: translate(-50%, -50%) rotate(0deg); } 100% { transform: translate(-50%, -50%) rotate(360deg); } }
        .modal-overlay { background-color: rgba(0, 0, 0, 0.5); backdrop-filter: blur(2px); }
        .tab-btn.active { border-bottom: 2px solid #2563eb; color: #2563eb; font-weight: 600; }
        .tab-btn { color: #6b7280; font-weight: 500; }

        /* Playlist Item Styles */
        .playlist-item { transition: all 0.2s; border-left: 3px solid transparent; }
        .playlist-item:hover { background-color: #f8fafc; }
        .playlist-item.active { background-color: #eff6ff; border-left-color: #2563eb; }
        .playlist-item.active .title { color: #1d4ed8; font-weight: 600; }
        .playlist-item.active .icon-play { opacity: 1; }
    </style>
</head>
<body class="bg-gray-100 h-screen overflow-hidden flex text-gray-800">

    <!-- Loading Overlay Fullscreen (Initial Load) -->
    <div id="app-loading" class="fixed inset-0 z-[60] bg-white flex flex-col items-center justify-center">
        <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-blue-600 mb-4"></div>
        <p class="text-gray-600 font-medium">Loading...</p>
    </div>

    <div id="mobile-overlay" onclick="toggleSidebar()" class="fixed inset-0 bg-black opacity-50 z-20 hidden lg:hidden"></div>

    <!-- Password Modal -->
    <div id="password-modal" class="fixed inset-0 z-50 hidden flex items-center justify-center modal-overlay">
        <div class="bg-white rounded-xl shadow-2xl p-6 w-full max-w-sm mx-4">
            <div class="text-center mb-6">
                <div class="w-16 h-16 bg-blue-100 rounded-full flex items-center justify-center mx-auto mb-4 text-blue-600">
                    <i class="fa-solid fa-lock text-2xl"></i>
                </div>
                <h3 class="text-xl font-bold text-gray-800">Akses Admin</h3>
                <p class="text-sm text-gray-500 mt-1">Masukkan password untuk pengaturan.</p>
            </div>
            <form onsubmit="verifyPassword(event)">
                <div class="mb-4">
                    <input type="password" id="admin-password" class="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none" placeholder="Password..." required>
                </div>
                <div class="flex gap-3">
                    <button type="button" onclick="closePasswordModal()" class="flex-1 px-4 py-2 bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200">Batal</button>
                    <button type="submit" class="flex-1 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 shadow-lg shadow-blue-500/30">Masuk</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Sidebar -->
    <aside id="sidebar" class="bg-slate-900 text-slate-300 w-64 flex-shrink-0 fixed inset-y-0 left-0 z-30 transform -translate-x-full lg:relative lg:translate-x-0 sidebar-transition flex flex-col h-full shadow-xl">
        <div class="h-16 flex items-center justify-center border-b border-slate-700 bg-slate-900 shadow-sm">
            <div class="flex items-center gap-3">
                <div class="bg-blue-600 text-white p-1.5 rounded-lg">
                    <i class="fa-solid fa-graduation-cap text-xl"></i>
                </div>
                <h1 class="text-xl font-bold text-white tracking-wide">NusanCBT</h1>
            </div>
        </div>
        <div class="p-4 border-b border-slate-700 bg-slate-800/50">
            <div class="flex items-center gap-3 mb-3">
                <img src="https://api.dicebear.com/7.x/avataaars/svg?seed=Felix" alt="User Avatar" class="w-10 h-10 rounded-full bg-slate-600 border-2 border-slate-500">
                <div>
                    <p class="text-sm font-semibold text-white">Siswa Peserta</p>
                    <p class="text-xs text-slate-400">Kelas TJKT</p>
                </div>
            </div>
        </div>
        <nav class="flex-1 overflow-y-auto custom-scrollbar p-3 space-y-1">
            <div class="text-xs font-semibold text-slate-500 uppercase tracking-wider mb-2 mt-2 px-3">Menu Utama</div>
            <button onclick="switchView('home')" id="btn-home" class="nav-item w-full flex items-center gap-3 px-3 py-3 rounded-lg hover:bg-slate-800 transition-colors active">
                <i class="fa-solid fa-house w-5 text-center text-slate-400 transition-colors"></i>
                <span class="text-sm font-medium">Dashboard</span>
            </button>
            <div id="sidebar-dynamic-menu"></div>
            <div class="text-xs font-semibold text-slate-500 uppercase tracking-wider mb-2 mt-6 px-3">Lainnya</div>
            <button onclick="switchView('settings')" id="btn-settings" class="nav-item w-full flex items-center gap-3 px-3 py-3 rounded-lg hover:bg-slate-800 transition-colors">
                <i class="fa-solid fa-gear w-5 text-center text-slate-400 transition-colors"></i>
                <span class="text-sm font-medium">Pengaturan</span>
            </button>
            <button class="nav-item w-full flex items-center gap-3 px-3 py-3 rounded-lg hover:bg-slate-800 text-red-400 hover:text-red-300 transition-colors mt-auto">
                <i class="fa-solid fa-right-from-bracket w-5 text-center"></i>
                <span class="text-sm font-medium">Keluar</span>
            </button>
        </nav>
    </aside>

    <!-- Main Content -->
    <main class="flex-1 flex flex-col h-full relative w-full">
        <header class="h-16 bg-white border-b border-gray-200 flex items-center justify-between px-4 lg:px-6 shadow-sm z-10">
            <div class="flex items-center gap-3">
                <button onclick="toggleSidebar()" class="lg:hidden p-2 text-gray-600 hover:bg-gray-100 rounded-md">
                    <i class="fa-solid fa-bars text-xl"></i>
                </button>
                <h2 id="page-title" class="text-lg font-bold text-gray-800">Dashboard Utama</h2>
            </div>
            <div class="flex items-center gap-4">
                <div class="hidden md:flex flex-col items-end mr-2">
                    <span id="digital-clock" class="text-sm font-mono font-bold text-blue-600">00:00:00</span>
                    <span id="current-date" class="text-xs text-gray-500">Senin, 1 Jan 2024</span>
                </div>
                <button onclick="toggleFullscreen()" class="p-2 text-gray-500 hover:text-blue-600 hover:bg-blue-50 rounded-full" title="Layar Penuh"><i id="fullscreen-icon" class="fa-solid fa-expand"></i></button>
            </div>
        </header>

        <div id="content-area" class="flex-1 p-0 overflow-hidden relative bg-gray-50">
            <div id="loader" class="loader"></div>

            <div id="view-home" class="h-full overflow-y-auto p-4 lg:p-6 animate-fade-in">
                <!-- Banner Penutupan/Akses Otomatis -->
                <div id="curfew-banner" class="hidden mb-6 bg-yellow-50 border-l-4 border-yellow-500 p-4 rounded-r shadow-sm transition-all duration-300">
                    <div class="flex items-center">
                        <div class="flex-shrink-0 text-yellow-500">
                            <i class="fa-solid fa-clock text-2xl"></i>
                        </div>
                        <div class="ml-3">
                            <h3 class="text-sm font-bold text-yellow-800">Info Akses Menu</h3>
                            <p class="text-sm text-yellow-700 mt-1">
                                Beberapa menu belum dibuka atau jadwal telah berakhir.
                            </p>
                        </div>
                    </div>
                </div>

                <div id="home-cards-container" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6 mb-6"></div>
                <div class="bg-white rounded-xl shadow-sm border border-gray-200 p-6">
                    <h3 class="text-lg font-bold text-gray-800 mb-4 border-b pb-2 flex justify-between items-center">
                        Pengumuman Sekolah <span class="text-xs font-normal text-gray-500 bg-gray-100 px-2 py-1 rounded">Terbaru</span>
                    </h3>
                    <div id="announcement-list" class="space-y-4"></div>
                </div>
            </div>

            <!-- PLAYER WRAPPER (Responsive Flex Column on Mobile, Row on Desktop) -->
            <div id="view-iframe-wrapper" class="hidden w-full h-full bg-white flex flex-col md:flex-row relative">
                
                <!-- 1. VIDEO CONTAINER -->
                <div id="video-container" class="relative flex-1 bg-black h-[50vh] md:h-full transition-all duration-300">
                    <iframe id="main-iframe" class="w-full h-full" src="" allowfullscreen allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"></iframe>
                    
                    <!-- Tombol Toggle Playlist Floating -->
                    <button id="playlist-toggle-btn" onclick="togglePlaylist()" class="hidden absolute top-4 right-4 z-20 bg-black/70 hover:bg-blue-600 text-white px-3 py-2 rounded-md shadow-lg border border-white/20 backdrop-blur-sm transition-all text-xs flex items-center gap-2 group">
                        <i id="playlist-toggle-icon" class="fa-solid fa-list"></i>
                        <span class="hidden group-hover:inline font-medium">Daftar Materi</span>
                    </button>

                    <div id="iframe-error" class="hidden absolute inset-0 flex flex-col items-center justify-center bg-gray-50 p-6 text-center">
                        <i class="fa-solid fa-face-frown text-6xl text-gray-300 mb-4"></i>
                        <h3 class="text-xl font-bold text-gray-700">Tidak dapat memuat halaman</h3>
                        <p class="text-sm text-gray-500 mt-2 max-w-md mx-auto">Mungkin website menolak ditampilkan dalam frame (X-Frame-Options) atau link rusak.</p>
                        <a id="external-link" href="#" target="_blank" class="mt-4 bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700">Buka di Tab Baru</a>
                    </div>
                </div>

                <!-- 2. PLAYLIST SIDEBAR -->
                <div id="playlist-sidebar" class="hidden w-full md:w-80 h-[50vh] md:h-full bg-white border-l border-gray-200 flex flex-col shadow-xl z-10 transition-all duration-300">
                    <div class="p-3 border-b border-gray-200 bg-gray-50 flex justify-between items-center">
                        <h3 class="font-bold text-gray-700 text-sm flex items-center">
                            <i class="fa-solid fa-list-ul mr-2 text-blue-600"></i> Materi Pembelajaran
                        </h3>
                        <div class="flex items-center gap-2">
                            <span id="playlist-counter" class="text-[10px] bg-blue-100 text-blue-700 px-2 py-0.5 rounded-full font-bold">0/0</span>
                            <button onclick="togglePlaylist()" class="text-gray-400 hover:text-red-500 md:hidden"><i class="fa-solid fa-xmark"></i></button>
                        </div>
                    </div>
                    <!-- List Container -->
                    <div id="playlist-items" class="flex-1 overflow-y-auto playlist-scroll p-0 bg-white">
                        <!-- Items injected by JS -->
                    </div>
                </div>
            </div>

            <div id="view-settings" class="hidden h-full overflow-y-auto p-4 lg:p-6 bg-gray-50">
                <!-- UPDATED: LEBAR KONTAINER SETTINGS JADI LEBIH LUAS (max-w-7xl) -->
                <div class="max-w-7xl mx-auto">
                    <div class="flex border-b border-gray-200 mb-6 bg-white rounded-t-xl px-4 pt-4 shadow-sm">
                        <button onclick="switchSettingsTab('menu')" id="tab-btn-menu" class="tab-btn active px-6 py-3 mr-2 focus:outline-none"><i class="fa-solid fa-list-check mr-2"></i> Menu & Link</button>
                        <button onclick="switchSettingsTab('announcement')" id="tab-btn-announcement" class="tab-btn px-6 py-3 mr-2 focus:outline-none"><i class="fa-solid fa-bullhorn mr-2"></i> Pengumuman</button>
                    </div>

                    <div id="tab-content-menu" class="space-y-6">
                        <div class="bg-white rounded-xl shadow-md p-6 border border-gray-200">
                            <div class="flex justify-between items-center mb-4">
                                <h3 class="text-lg font-bold text-gray-800">Editor Menu</h3>
                                <button onclick="resetMenuForm()" class="text-sm text-gray-500 hover:text-blue-600"><i class="fa-solid fa-rotate-left mr-1"></i> Reset</button>
                            </div>
                            <form onsubmit="saveMenu(event)" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                <input type="hidden" id="menu-edit-id">
                                <div class="col-span-1"><label class="block text-sm font-medium text-gray-700 mb-1">Nama Menu</label><input type="text" id="menu-title" class="w-full px-4 py-2 rounded-lg border border-gray-300 outline-none" placeholder="Cth: Video Pembelajaran" required></div>
                                <div class="col-span-1"><label class="block text-sm font-medium text-gray-700 mb-1">Link URL</label><input type="text" id="menu-url" class="w-full px-4 py-2 rounded-lg border border-gray-300 outline-none" placeholder="https://youtube.com/..." required></div>
                                <div><label class="block text-sm font-medium text-gray-700 mb-1">Ikon</label><select id="menu-icon" class="w-full px-4 py-2 rounded-lg border border-gray-300 outline-none"><option value="fa-desktop">Desktop</option><option value="fa-book-open">Buku</option><option value="fa-pen-to-square">Tulis</option><option value="fa-video">Video</option><option value="fa-file-pdf">Dokumen</option><option value="fa-globe">Bola Dunia</option></select></div>
                                <div><label class="block text-sm font-medium text-gray-700 mb-1">Warna</label><select id="menu-color" class="w-full px-4 py-2 rounded-lg border border-gray-300 outline-none"><option value="blue">Biru</option><option value="green">Hijau</option><option value="purple">Ungu</option><option value="orange">Oranye</option><option value="red">Merah</option></select></div>
                                
                                <!-- INPUT STATUS -->
                                <div class="md:col-span-2"><label class="block text-sm font-medium text-gray-700 mb-1">Status Menu (Manual)</label><select id="menu-status" class="w-full px-4 py-2 rounded-lg border border-gray-300 outline-none"><option value="active">Aktif</option><option value="inactive">Non-Aktif (Sembunyikan Manual)</option></select></div>

                                <!-- INPUT JAM BUKA & TUTUP -->
                                <div class="md:col-span-1">
                                    <label class="block text-sm font-medium text-gray-700 mb-1">Jam Buka (Jam:Menit)</label>
                                    <input type="time" id="menu-auto-open" class="w-full px-4 py-2 rounded-lg border border-gray-300 outline-none">
                                </div>
                                <div class="md:col-span-1">
                                    <label class="block text-sm font-medium text-gray-700 mb-1">Jam Tutup (Jam:Menit)</label>
                                    <input type="time" id="menu-auto-close" class="w-full px-4 py-2 rounded-lg border border-gray-300 outline-none">
                                </div>
                                <div class="md:col-span-2">
                                    <p class="text-xs text-gray-400">Biarkan kosong jika ingin menu selalu aktif (tanpa jadwal).</p>
                                </div>

                                <div class="md:col-span-2"><button type="submit" id="btn-save-menu" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded-lg transition-colors shadow"><i class="fa-solid fa-plus mr-2"></i> Simpan ke Database</button></div>
                            </form>
                            <div class="mt-4 p-3 bg-blue-50 text-blue-700 text-sm rounded border border-blue-100">
                                <i class="fa-solid fa-list-ul mr-2"></i> <strong>Fitur Playlist Youtube:</strong>
                                <p class="mt-1 ml-6 text-xs">Untuk membuat playlist video, pisahkan link dengan tanda koma ( , ).<br>Contoh: <code>youtube.com/v1, youtube.com/v2</code></p>
                            </div>
                        </div>
                        <div class="bg-white rounded-xl shadow-md border border-gray-200 overflow-hidden">
                            <div class="px-6 py-4 border-b border-gray-200 bg-gray-50 flex justify-between items-center"><h3 class="font-bold text-gray-700">Daftar Menu Aktif</h3><button onclick="fetchFromDatabase(false)" class="text-xs text-blue-500 hover:text-blue-700 underline"><i class="fa-solid fa-rotate mr-1"></i>Refresh Data</button></div>
                            <div id="settings-menu-list" class="divide-y divide-gray-100"></div>
                        </div>
                    </div>

                    <div id="tab-content-announcement" class="hidden space-y-6">
                        <div class="bg-white rounded-xl shadow-md p-6 border border-gray-200">
                            <h3 class="text-lg font-bold text-gray-800 mb-4">Buat Pengumuman Baru</h3>
                            <form onsubmit="saveAnnouncement(event)" class="space-y-4">
                                <input type="hidden" id="ann-edit-id">
                                <div><label class="block text-sm font-medium text-gray-700 mb-1">Judul</label><input type="text" id="ann-title" class="w-full px-4 py-2 rounded-lg border border-gray-300 outline-none" required></div>
                                <div><label class="
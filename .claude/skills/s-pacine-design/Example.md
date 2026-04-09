# Pa' Cine -- Example Home Page

A simplified, self-contained version of the home page showing the Header, tab bar, movie grid, loading/empty states, and Footer all in one file. Use this as a reference when creating new pages.

```tsx
"use client";

import { useState } from "react";
import Link from "next/link";
import Image from "next/image";
import { Menu, Film, Building2 } from "lucide-react";

// ---------------------------------------------------------------------------
// Sample data (replace with real data fetching)
// ---------------------------------------------------------------------------
const SAMPLE_MOVIES = [
  { slug: "el-brutalista", title_es: "El Brutalista", poster_url: "/posters/brutalista.jpg" },
  { slug: "anora", title_es: "Anora", poster_url: "/posters/anora.jpg" },
  { slug: "emilia-perez", title_es: "Emilia Perez", poster_url: "/posters/emilia.jpg" },
  { slug: "la-sustancia", title_es: "La Sustancia", poster_url: "/posters/sustancia.jpg" },
  { slug: "conclave", title_es: "Conclave", poster_url: "/posters/conclave.jpg" },
  { slug: "wicked", title_es: "Wicked", poster_url: "/posters/wicked.jpg" },
];

const COMING_SOON = [
  { slug: "superman", title_es: "Superman", poster_url: "/posters/superman.jpg" },
  { slug: "thunderbolts", title_es: "Thunderbolts*", poster_url: "/posters/thunderbolts.jpg" },
];

// ---------------------------------------------------------------------------
// Header
// ---------------------------------------------------------------------------
function Header() {
  const [menuOpen, setMenuOpen] = useState(false);

  return (
    <header className="bg-zinc-900">
      <div className="max-w-7xl mx-auto px-4 h-12 flex items-center justify-between">
        <Link href="/" className="flex items-center gap-2 hover:opacity-80 transition-opacity">
          {/* Three red stripes logo */}
          <div className="flex items-center gap-0.5">
            <div className="w-2 h-8 bg-red-500 rounded-sm" />
            <div className="w-2 h-8 bg-red-500 rounded-sm" />
            <div className="w-2 h-8 bg-red-500 rounded-sm" />
          </div>
          <div>
            <div className="text-xl font-black tracking-tight leading-none text-white">
              PA&apos;<span className="text-red-500">CINE</span>
            </div>
            <div className="text-[10px] text-white tracking-wider uppercase">
              El cine en Colombia
            </div>
          </div>
        </Link>

        <button
          type="button"
          aria-label="Abrir menú"
          onClick={() => setMenuOpen(!menuOpen)}
          className="p-2 text-white hover:text-zinc-300 transition-colors"
        >
          <Menu className="h-6 w-6" />
        </button>
      </div>

      {menuOpen && (
        <nav className="bg-zinc-900 border-t border-zinc-700 px-4 py-2 flex flex-col gap-1">
          <Link href="/" onClick={() => setMenuOpen(false)}
            className="flex items-center gap-3 px-3 py-2.5 rounded-lg text-white hover:bg-zinc-800 transition-colors">
            <Film className="h-5 w-5" /> Cartelera
          </Link>
          <Link href="/all_cines" onClick={() => setMenuOpen(false)}
            className="flex items-center gap-3 px-3 py-2.5 rounded-lg text-white hover:bg-zinc-800 transition-colors">
            <Building2 className="h-5 w-5" /> Todos los cines
          </Link>
        </nav>
      )}
    </header>
  );
}

// ---------------------------------------------------------------------------
// Footer
// ---------------------------------------------------------------------------
function Footer() {
  return (
    <footer className="bg-zinc-900 text-zinc-400 mt-auto">
      <div className="max-w-7xl mx-auto px-4 py-6">
        <div className="flex flex-col sm:flex-row items-center justify-between gap-4 text-sm">
          <p>
            Hecho con{" "}
            <svg viewBox="0 0 64 64" width="1.95em" height="1.95em"
              style={{ verticalAlign: "-0.35em", display: "inline" }}>
              <defs>
                <clipPath id="heartClip">
                  <path d="M32 56s-18-11.8-25.2-21.4C1.6 27.6 4 18.5 11.2 14.8c6.2-3.2 13.3-1.1 17 4.2
                    3.7-5.3 10.8-7.4 17-4.2C60 18.5 62.4 27.6 57.2 34.6 50 44.2 32 56 32 56z" />
                </clipPath>
              </defs>
              <g clipPath="url(#heartClip)">
                <rect x="0" y="0" width="64" height="26" fill="#FCD116" />
                <rect x="0" y="26" width="64" height="16" fill="#003893" />
                <rect x="0" y="42" width="64" height="22" fill="#CE1126" />
              </g>
            </svg>
            {" "}en Colombia
          </p>
          <p>&copy; {new Date().getFullYear()} Pa&apos; Cine</p>
          <nav className="flex gap-6">
            <Link href="/privacidad" className="hover:text-white transition-colors">Privacidad</Link>
            <Link href="/terminos" className="hover:text-white transition-colors">Términos</Link>
          </nav>
        </div>
      </div>
    </footer>
  );
}

// ---------------------------------------------------------------------------
// MovieCard
// ---------------------------------------------------------------------------
function MovieCard({ movie }: { movie: { slug: string; title_es: string; poster_url: string } }) {
  const [loaded, setLoaded] = useState(false);
  const [error, setError] = useState(false);

  return (
    <div className="card-base overflow-hidden hover:shadow-md transition-shadow">
      <Link href={`/movies/${movie.slug}`}>
        <div className="aspect-[2/3] relative overflow-hidden bg-zinc-200">
          {movie.poster_url && !error ? (
            <>
              {!loaded && <div className="absolute inset-0 bg-zinc-200" />}
              <Image
                src={movie.poster_url}
                alt={movie.title_es}
                fill
                sizes="(max-width: 768px) 33vw, 16vw"
                onLoad={() => setLoaded(true)}
                onError={() => setError(true)}
                className={`object-cover hover:scale-105 transition-transform duration-300 ${
                  loaded ? "opacity-100" : "opacity-0"
                }`}
              />
            </>
          ) : (
            <div className="w-full h-full flex items-center justify-center text-zinc-400 text-xs sm:text-base">
              Sin imagen
            </div>
          )}
        </div>
        <div className="px-2 py-1 sm:px-4 sm:py-2">
          <h3 className="font-semibold text-xs sm:text-base lg:text-lg text-zinc-900 mb-0.5 sm:mb-1 hover:text-red-500 transition-colors truncate whitespace-nowrap">
            {movie.title_es}
          </h3>
        </div>
      </Link>
    </div>
  );
}

// ---------------------------------------------------------------------------
// Loading Spinner
// ---------------------------------------------------------------------------
function LoadingSpinner() {
  return (
    <div className="flex flex-col items-center justify-center py-20">
      <div className="h-10 w-10 animate-spin rounded-full border-4 border-zinc-300 border-t-red-500" />
      <p className="mt-4 text-sm text-zinc-500">Cargando cartelera...</p>
    </div>
  );
}

// ---------------------------------------------------------------------------
// Empty State
// ---------------------------------------------------------------------------
function EmptyState() {
  return (
    <div className="text-center py-16">
      <p className="text-zinc-500 text-lg">
        No hay películas en cartelera en este momento.
      </p>
    </div>
  );
}

// ---------------------------------------------------------------------------
// Home Page
// ---------------------------------------------------------------------------
export default function HomePage() {
  const [activeTab, setActiveTab] = useState<"cartelera" | "cines">("cartelera");
  const isLoading = false; // toggle to true to see spinner
  const isEmpty = false;   // toggle to true to see empty state

  const totalMovies = SAMPLE_MOVIES.length + COMING_SOON.length;

  return (
    <div className="min-h-screen bg-zinc-100 flex flex-col">
      <Header />

      <main className="max-w-7xl mx-auto px-0 md:px-4 py-2 flex-1 w-full">
        {/* Tab bar */}
        <div className="flex items-center gap-3 flex-wrap mb-1 md:mb-4 px-4 md:px-0">
          <div role="tablist" className="flex items-center gap-4 pb-2">
            <button
              type="button"
              role="tab"
              aria-selected={activeTab === "cartelera"}
              onClick={() => setActiveTab("cartelera")}
              className={`pb-1 text-2xl font-black transition-colors border-b-[3px] ${
                activeTab === "cartelera"
                  ? "text-zinc-900 border-red-500"
                  : "text-zinc-300 border-transparent hover:text-zinc-500"
              }`}
            >
              Cartelera
            </button>
            <button
              type="button"
              role="tab"
              aria-selected={activeTab === "cines"}
              onClick={() => setActiveTab("cines")}
              className={`pb-1 text-2xl font-black transition-colors border-b-[3px] ${
                activeTab === "cines"
                  ? "text-zinc-900 border-red-500"
                  : "text-zinc-300 border-transparent hover:text-zinc-500"
              }`}
            >
              Cines
            </button>
          </div>

          {/* Movie count (desktop) */}
          <div className="hidden md:flex items-center gap-1 ml-auto text-sm text-zinc-500">
            {totalMovies} película{totalMovies !== 1 && "s"} cerca de{" "}
            <button className="underline hover:text-zinc-700 transition-colors">
              Medellín
            </button>
          </div>
        </div>

        {/* Movie count (mobile) */}
        <div className="md:hidden px-4 mb-3">
          <p className="text-sm text-zinc-500">
            {totalMovies} película{totalMovies !== 1 && "s"} cerca de{" "}
            <button className="underline hover:text-zinc-700 transition-colors">
              Medellín
            </button>
          </p>
        </div>

        {/* Tab content */}
        {activeTab === "cartelera" && (
          <div className="px-4 md:px-0">
            {isLoading ? (
              <LoadingSpinner />
            ) : isEmpty ? (
              <EmptyState />
            ) : (
              <>
                {/* Now Showing */}
                <section className="mb-8">
                  <div className="grid grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6 gap-3 sm:gap-4 lg:gap-6">
                    {SAMPLE_MOVIES.map((movie) => (
                      <MovieCard key={movie.slug} movie={movie} />
                    ))}
                  </div>
                </section>

                {/* Coming Soon */}
                <section>
                  <h2 className="text-lg font-semibold text-zinc-700 mb-3">
                    Pronto ({COMING_SOON.length} películas)
                  </h2>
                  <div className="grid grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6 gap-3 sm:gap-4 lg:gap-6">
                    {COMING_SOON.map((movie) => (
                      <MovieCard key={movie.slug} movie={movie} />
                    ))}
                  </div>
                </section>
              </>
            )}
          </div>
        )}

        {activeTab === "cines" && (
          <div className="px-4 md:px-0">
            <p className="text-zinc-500 text-lg py-16 text-center">
              Lista de cines iría aquí.
            </p>
          </div>
        )}
      </main>

      <Footer />
    </div>
  );
}
```

## What this example demonstrates

1. **Page shell** -- `min-h-screen bg-zinc-100 flex flex-col` with Header/Footer
2. **Header** -- Dark navbar with red stripe logo, hamburger menu, nav links
3. **Footer** -- Colombian flag heart, copyright, legal links
4. **Tab bar** -- Bold tabs with red underline for active state
5. **Movie grid** -- Responsive columns (3/4/5/6) with consistent gaps
6. **MovieCard** -- `.card-base`, poster with `aspect-[2/3]`, image hover zoom, title hover red
7. **Loading spinner** -- Zinc/red spinning circle with text
8. **Empty state** -- Centered muted text
9. **Section headings** -- `text-lg font-semibold text-zinc-700`
10. **Responsive summary text** -- Hidden/shown at different breakpoints

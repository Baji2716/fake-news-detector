import React, { useState } from "react";
import { Link, useLocation } from "react-router-dom";
import { createPageUrl } from "@/utils";
import { motion, AnimatePresence } from "framer-motion";
import { Shield, Search, LayoutDashboard, Clock, BookOpen, Menu, X } from "lucide-react";

const navItems = [
  { name: "Dashboard", icon: LayoutDashboard, page: "Dashboard" },
  { name: "Analyze", icon: Search, page: "Analyze" },
  { name: "History", icon: Clock, page: "History" },
  { name: "Learn", icon: BookOpen, page: "Learn" },
];

export default function Layout({ children, currentPageName }) {
  const [mobileOpen, setMobileOpen] = useState(false);

  return (
    <div className="min-h-screen bg-[#0a0e1a] text-white">
      <style>{`
        :root {
          --background: 222 47% 6%;
          --foreground: 210 40% 96%;
          --card: 222 47% 8%;
          --card-foreground: 210 40% 96%;
          --popover: 222 47% 8%;
          --popover-foreground: 210 40% 96%;
          --primary: 217 91% 60%;
          --primary-foreground: 222 47% 6%;
          --secondary: 217 33% 17%;
          --secondary-foreground: 210 40% 96%;
          --muted: 217 33% 17%;
          --muted-foreground: 215 20% 55%;
          --accent: 217 33% 17%;
          --accent-foreground: 210 40% 96%;
          --destructive: 0 84% 60%;
          --destructive-foreground: 210 40% 96%;
          --border: 217 33% 17%;
          --input: 217 33% 17%;
          --ring: 217 91% 60%;
          --radius: 0.75rem;
        }
        body {
          background: #0a0e1a;
        }
      `}</style>

      {/* Ambient glow */}
      <div className="fixed inset-0 pointer-events-none overflow-hidden">
        <div className="absolute top-[-20%] left-[-10%] w-[500px] h-[500px] rounded-full bg-blue-600/[0.04] blur-[120px]" />
        <div className="absolute bottom-[-20%] right-[-10%] w-[500px] h-[500px] rounded-full bg-violet-600/[0.04] blur-[120px]" />
      </div>

      {/* Top Nav */}
      <nav className="sticky top-0 z-50 border-b border-white/[0.06] bg-[#0a0e1a]/80 backdrop-blur-xl">
        <div className="max-w-7xl mx-auto px-4 h-16 flex items-center justify-between">
          <Link to={createPageUrl("Dashboard")} className="flex items-center gap-2.5">
            <div className="w-9 h-9 rounded-xl bg-gradient-to-br from-blue-500 to-violet-600 flex items-center justify-center">
              <Shield className="w-5 h-5 text-white" />
            </div>
            <div>
              <span className="text-base font-bold text-white tracking-tight">TruthLens</span>
              <span className="text-[10px] text-slate-500 block -mt-0.5">Fake News Detector</span>
            </div>
          </Link>

          {/* Desktop Nav */}
          <div className="hidden md:flex items-center gap-1">
            {navItems.map((item) => {
              const isActive = currentPageName === item.page;
              return (
                <Link key={item.page} to={createPageUrl(item.page)}>
                  <div className={`relative px-4 py-2 rounded-xl flex items-center gap-2 text-sm font-medium transition-all duration-200 ${
                    isActive
                      ? "text-white"
                      : "text-slate-400 hover:text-slate-200 hover:bg-white/[0.04]"
                  }`}>
                    <item.icon className="w-4 h-4" />
                    {item.name}
                    {isActive && (
                      <motion.div
                        layoutId="activeNav"
                        className="absolute inset-0 bg-white/[0.06] rounded-xl border border-white/[0.08]"
                        style={{ zIndex: -1 }}
                        transition={{ type: "spring", bounce: 0.2, duration: 0.5 }}
                      />
                    )}
                  </div>
                </Link>
              );
            })}
          </div>

          {/* Mobile Menu Button */}
          <button
            onClick={() => setMobileOpen(!mobileOpen)}
            className="md:hidden p-2 rounded-xl text-slate-400 hover:text-white hover:bg-white/[0.05] transition-colors"
          >
            {mobileOpen ? <X className="w-5 h-5" /> : <Menu className="w-5 h-5" />}
          </button>
        </div>

        {/* Mobile Nav */}
        <AnimatePresence>
          {mobileOpen && (
            <motion.div
              initial={{ height: 0, opacity: 0 }}
              animate={{ height: "auto", opacity: 1 }}
              exit={{ height: 0, opacity: 0 }}
              className="md:hidden overflow-hidden border-t border-white/[0.06]"
            >
              <div className="p-3 space-y-1">
                {navItems.map((item) => {
                  const isActive = currentPageName === item.page;
                  return (
                    <Link
                      key={item.page}
                      to={createPageUrl(item.page)}
                      onClick={() => setMobileOpen(false)}
                    >
                      <div className={`px-4 py-3 rounded-xl flex items-center gap-3 text-sm font-medium transition-all ${
                        isActive
                          ? "text-white bg-white/[0.06]"
                          : "text-slate-400 hover:text-white hover:bg-white/[0.04]"
                      }`}>
                        <item.icon className="w-4 h-4" />
                        {item.name}
                      </div>
                    </Link>
                  );
                })}
              </div>
            </motion.div>
          )}
        </AnimatePresence>
      </nav>

      {/* Content */}
      <main className="relative z-10">
        {children}
      </main>
    </div>
  );
}

import React from "react";
import { base44 } from "@/api/base44Client";
import { useQuery } from "@tanstack/react-query";
import { motion } from "framer-motion";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { FileSearch, Shield, AlertTriangle, TrendingUp, BarChart3 } from "lucide-react";
import { Link } from "react-router-dom";
import { createPageUrl } from "@/utils";
import StatCard from "@/components/dashboard/StatCard";
import VerdictChart from "@/components/dashboard/VerdictChart";
import HistoryCard from "@/components/history/HistoryCard";

export default function Dashboard() {
  const { data: analyses = [], isLoading } = useQuery({
    queryKey: ["analyses"],
    queryFn: () => base44.entities.Analysis.list("-created_date", 50),
  });

  const totalAnalyses = analyses.length;
  const avgScore = totalAnalyses > 0
    ? Math.round(analyses.reduce((s, a) => s + (a.credibility_score || 0), 0) / totalAnalyses)
    : 0;
  const totalRedFlags = analyses.reduce((s, a) => s + (a.red_flags?.length || 0), 0);
  const reliableCount = analyses.filter((a) => a.verdict === "reliable" || a.verdict === "mostly_reliable").length;

  const recentAnalyses = analyses.slice(0, 5);

  return (
    <div className="min-h-screen p-4 md:p-8">
      <div className="max-w-6xl mx-auto">
        <motion.div initial={{ opacity: 0, y: -20 }} animate={{ opacity: 1, y: 0 }}>
          <h1 className="text-3xl font-bold text-white mb-1">Dashboard</h1>
          <p className="text-slate-400 text-sm mb-8">Your misinformation detection overview</p>
        </motion.div>

        {/* Stats */}
        <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-8">
          <StatCard title="Total Analyses" value={totalAnalyses} icon={FileSearch} color="bg-blue-500/10 text-blue-400" index={0} />
          <StatCard title="Avg. Score" value={avgScore} subtitle="out of 100" icon={TrendingUp} color="bg-emerald-500/10 text-emerald-400" index={1} />
          <StatCard title="Red Flags" value={totalRedFlags} subtitle="detected total" icon={AlertTriangle} color="bg-red-500/10 text-red-400" index={2} />
          <StatCard title="Reliable" value={reliableCount} subtitle={`of ${totalAnalyses} articles`} icon={Shield} color="bg-violet-500/10 text-violet-400" index={3} />
        </div>

        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          {/* Chart */}
          <Card className="bg-white/[0.03] border-white/[0.06] md:col-span-1">
            <CardHeader className="pb-2">
              <CardTitle className="text-base font-semibold text-slate-200 flex items-center gap-2">
                <BarChart3 className="w-4 h-4 text-violet-400" />
                Verdict Breakdown
              </CardTitle>
            </CardHeader>
            <CardContent>
              <VerdictChart analyses={analyses} />
            </CardContent>
          </Card>

          {/* Recent */}
          <div className="md:col-span-2 space-y-3">
            <div className="flex items-center justify-between">
              <h2 className="text-base font-semibold text-slate-200">Recent Analyses</h2>
              <Link to={createPageUrl("History")} className="text-xs text-blue-400 hover:text-blue-300 transition-colors">
                View all →
              </Link>
            </div>
            {isLoading ? (
              <div className="space-y-3">
                {[0, 1, 2].map((i) => (
                  <div key={i} className="h-24 rounded-2xl bg-white/[0.03] animate-pulse" />
                ))}
              </div>
            ) : recentAnalyses.length === 0 ? (
              <div className="text-center py-16 text-slate-500 text-sm">
                No analyses yet. Go to Analyze to check your first article!
              </div>
            ) : (
              <div className="space-y-3">
                {recentAnalyses.map((a, i) => (
                  <HistoryCard key={a.id} analysis={a} index={i} />
                ))}
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}

import React, { useState } from "react";
import { base44 } from "@/api/base44Client";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Input } from "@/components/ui/input";
import { motion, AnimatePresence } from "framer-motion";
import { Search, Loader2, Sparkles, Link as LinkIcon } from "lucide-react";
import AnalysisResults from "@/components/analyze/AnalysisResults";

export default function Analyze() {
  const [articleText, setArticleText] = useState("");
  const [articleUrl, setArticleUrl] = useState("");
  const [isAnalyzing, setIsAnalyzing] = useState(false);
  const [result, setResult] = useState(null);

  const handleAnalyze = async () => {
    if (!articleText.trim() && !articleUrl.trim()) return;
    setIsAnalyzing(true);
    setResult(null);

    const prompt = `You are an expert fact-checker and media literacy analyst. Analyze the following article for credibility and misinformation.

${articleUrl ? `Article URL: ${articleUrl}` : ""}
Article Text:
"""
${articleText}
"""

Provide a thorough analysis including:
1. A credibility score from 0-100
2. A verdict: "reliable", "mostly_reliable", "mixed", "mostly_unreliable", or "unreliable"
3. A concise, trustworthy summary of the actual facts (2-3 sentences)
4. Any red flags found (sensationalist language, lack of sources, logical fallacies, emotional manipulation, unverified claims, etc.)
5. Credibility factors breakdown (source quality, evidence strength, language objectivity, factual accuracy, consistency) each scored 0-100
6. Bias level: "none", "slight", "moderate", or "strong"
7. Fact-check notes with additional context

Be thorough but fair. Assess based on journalistic standards.`;

    const analysisData = await base44.integrations.Core.InvokeLLM({
      prompt,
      add_context_from_internet: true,
      response_json_schema: {
        type: "object",
        properties: {
          title: { type: "string", description: "A descriptive title for this article" },
          credibility_score: { type: "number" },
          verdict: { type: "string", enum: ["reliable", "mostly_reliable", "mixed", "mostly_unreliable", "unreliable"] },
          summary: { type: "string" },
          red_flags: {
            type: "array",
            items: {
              type: "object",
              properties: {
                flag: { type: "string" },
                explanation: { type: "string" }
              }
            }
          },
          credibility_factors: {
            type: "array",
            items: {
              type: "object",
              properties: {
                factor: { type: "string" },
                score: { type: "number" },
                detail: { type: "string" }
              }
            }
          },
          bias_detected: { type: "string", enum: ["none", "slight", "moderate", "strong"] },
          fact_check_notes: { type: "string" }
        }
      }
    });

    const saved = await base44.entities.Analysis.create({
      ...analysisData,
      source_url: articleUrl || undefined,
      original_text: articleText,
    });

    setResult(saved);
    setIsAnalyzing(false);
  };

  return (
    <div className="min-h-screen p-4 md:p-8">
      <div className="max-w-4xl mx-auto">
        {/* Hero */}
        <motion.div
          initial={{ opacity: 0, y: -20 }}
          animate={{ opacity: 1, y: 0 }}
          className="text-center mb-10"
        >
          <div className="inline-flex items-center gap-2 px-4 py-1.5 rounded-full bg-blue-500/10 border border-blue-500/20 text-blue-400 text-xs font-medium mb-4">
            <Sparkles className="w-3 h-3" />
            AI-Powered Analysis
          </div>
          <h1 className="text-3xl md:text-4xl font-bold text-white mb-3">
            Analyze Any Article
          </h1>
          <p className="text-slate-400 max-w-lg mx-auto text-sm">
            Paste an article or URL below. Our AI will assess its credibility, detect bias,
            flag misinformation, and provide a trustworthy summary.
          </p>
        </motion.div>

        {/* Input */}
        <motion.div
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.1 }}
          className="space-y-4 mb-10"
        >
          <div className="relative">
            <LinkIcon className="absolute left-4 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-500" />
            <Input
              placeholder="Paste article URL (optional)"
              value={articleUrl}
              onChange={(e) => setArticleUrl(e.target.value)}
              className="pl-11 bg-white/[0.03] border-white/[0.08] text-white placeholder:text-slate-500 h-12 rounded-xl focus:border-blue-500/50 focus:ring-blue-500/20"
            />
          </div>
          <Textarea
            placeholder="Paste the article text here..."
            value={articleText}
            onChange={(e) => setArticleText(e.target.value)}
            className="min-h-[200px] bg-white/[0.03] border-white/[0.08] text-white placeholder:text-slate-500 rounded-xl focus:border-blue-500/50 focus:ring-blue-500/20 resize-none"
          />
          <Button
            onClick={handleAnalyze}
            disabled={isAnalyzing || (!articleText.trim() && !articleUrl.trim())}
            className="w-full h-12 rounded-xl bg-gradient-to-r from-blue-600 to-violet-600 hover:from-blue-500 hover:to-violet-500 text-white font-semibold text-sm transition-all duration-300 disabled:opacity-40"
          >
            {isAnalyzing ? (
              <span className="flex items-center gap-2">
                <Loader2 className="w-4 h-4 animate-spin" />
                Analyzing article...
              </span>
            ) : (
              <span className="flex items-center gap-2">
                <Search className="w-4 h-4" />
                Analyze for Misinformation
              </span>
            )}
          </Button>
        </motion.div>

        {/* Loading State */}
        <AnimatePresence>
          {isAnalyzing && (
            <motion.div
              initial={{ opacity: 0 }}
              animate={{ opacity: 1 }}
              exit={{ opacity: 0 }}
              className="text-center py-16"
            >
              <div className="relative inline-block">
                <div className="w-16 h-16 rounded-full border-2 border-blue-500/20 border-t-blue-500 animate-spin" />
                <Sparkles className="w-5 h-5 text-blue-400 absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2" />
              </div>
              <p className="text-slate-400 mt-4 text-sm">
                Cross-referencing facts and checking sources...
              </p>
            </motion.div>
          )}
        </AnimatePresence>

        {/* Results */}
        {!isAnalyzing && result && <AnalysisResults result={result} />}
      </div>
    </div>
  );
}

import React, { useState } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery } from "@tanstack/react-query";
import { motion } from "framer-motion";
import { Input } from "@/components/ui/input";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Search, SlidersHorizontal } from "lucide-react";
import HistoryCard from "@/components/history/HistoryCard";

export default function History() {
  const [search, setSearch] = useState("");
  const [verdictFilter, setVerdictFilter] = useState("all");

  const { data: analyses = [], isLoading } = useQuery({
    queryKey: ["analyses-history"],
    queryFn: () => base44.entities.Analysis.list("-created_date", 100),
  });

  const filtered = analyses.filter((a) => {
    const matchSearch = !search || a.title?.toLowerCase().includes(search.toLowerCase()) || a.summary?.toLowerCase().includes(search.toLowerCase());
    const matchVerdict = verdictFilter === "all" || a.verdict === verdictFilter;
    return matchSearch && matchVerdict;
  });

  return (
    <div className="min-h-screen p-4 md:p-8">
      <div className="max-w-4xl mx-auto">
        <motion.div initial={{ opacity: 0, y: -20 }} animate={{ opacity: 1, y: 0 }}>
          <h1 className="text-3xl font-bold text-white mb-1">Analysis History</h1>
          <p className="text-slate-400 text-sm mb-8">Browse all your past analyses</p>
        </motion.div>

        {/* Filters */}
        <motion.div
          initial={{ opacity: 0, y: 10 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.1 }}
          className="flex flex-col sm:flex-row gap-3 mb-6"
        >
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-500" />
            <Input
              placeholder="Search articles..."
              value={search}
              onChange={(e) => setSearch(e.target.value)}
              className="pl-10 bg-white/[0.03] border-white/[0.08] text-white placeholder:text-slate-500 rounded-xl h-10"
            />
          </div>
          <div className="flex items-center gap-2">
            <SlidersHorizontal className="w-4 h-4 text-slate-500" />
            <Select value={verdictFilter} onValueChange={setVerdictFilter}>
              <SelectTrigger className="w-44 bg-white/[0.03] border-white/[0.08] text-slate-300 rounded-xl h-10">
                <SelectValue />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="all">All Verdicts</SelectItem>
                <SelectItem value="reliable">Reliable</SelectItem>
                <SelectItem value="mostly_reliable">Mostly Reliable</SelectItem>
                <SelectItem value="mixed">Mixed</SelectItem>
                <SelectItem value="mostly_unreliable">Mostly Unreliable</SelectItem>
                <SelectItem value="unreliable">Unreliable</SelectItem>
              </SelectContent>
            </Select>
          </div>
        </motion.div>

        {/* Results */}
        {isLoading ? (
          <div className="space-y-3">
            {[0, 1, 2, 3].map((i) => (
              <div key={i} className="h-24 rounded-2xl bg-white/[0.03] animate-pulse" />
            ))}
          </div>
        ) : filtered.length === 0 ? (
          <div className="text-center py-20">
            <p className="text-slate-500 text-sm">
              {analyses.length === 0 ? "No analyses yet." : "No results match your filters."}
            </p>
          </div>
        ) : (
          <div className="space-y-3">
            {filtered.map((a, i) => (
              <HistoryCard key={a.id} analysis={a} index={i} />
            ))}
          </div>
        )}
      </div>
    </div>
  );
}

import React from "react";
import { motion } from "framer-motion";
import { Card, CardContent } from "@/components/ui/card";
import {
  Eye, Quote, Globe, Users, BookOpen, BrainCircuit,
  CheckCircle, XCircle, AlertTriangle
} from "lucide-react";

const tips = [
  {
    icon: Eye,
    color: "text-blue-400 bg-blue-500/10",
    title: "Check the Source",
    description: "Always verify who published the article. Look for an 'About' page, check if the domain is a known news outlet, and see if real journalists are credited."
  },
  {
    icon: Quote,
    color: "text-violet-400 bg-violet-500/10",
    title: "Look for Citations",
    description: "Reliable articles cite their sources — studies, official reports, expert quotes. If there are no sources, that's a major red flag."
  },
  {
    icon: Globe,
    color: "text-emerald-400 bg-emerald-500/10",
    title: "Cross-Reference",
    description: "Search for the same story on multiple reputable news sites. If only one obscure site is reporting it, be very skeptical."
  },
  {
    icon: AlertTriangle,
    color: "text-amber-400 bg-amber-500/10",
    title: "Watch for Emotional Language",
    description: "Fake news often uses shocking headlines, ALL CAPS, and emotional manipulation. Real journalism uses measured, objective language."
  },
  {
    icon: Users,
    color: "text-pink-400 bg-pink-500/10",
    title: "Check the Author",
    description: "Search for the author's name. Do they have a track record? Have they written for reputable outlets? Anonymous authors are a warning sign."
  },
  {
    icon: BrainCircuit,
    color: "text-cyan-400 bg-cyan-500/10",
    title: "Beware of Confirmation Bias",
    description: "We tend to believe things that align with our existing views. Be extra critical of articles that feel too perfectly aligned with your opinions."
  },
];

const quickChecklist = [
  { check: true, text: "Article cites specific, verifiable sources" },
  { check: true, text: "Published by a recognized news organization" },
  { check: true, text: "Author has a verifiable track record" },
  { check: true, text: "Uses neutral, objective language" },
  { check: true, text: "Story confirmed by multiple outlets" },
  { check: false, text: "Uses ALL CAPS or excessive punctuation!!!" },
  { check: false, text: "Plays on strong emotions (fear, anger, outrage)" },
  { check: false, text: "Contains no sources or references" },
  { check: false, text: "URL looks suspicious or mimics a real site" },
  { check: false, text: "Date is missing or very old" },
];

export default function Learn() {
  return (
    <div className="min-h-screen p-4 md:p-8">
      <div className="max-w-4xl mx-auto">
        <motion.div initial={{ opacity: 0, y: -20 }} animate={{ opacity: 1, y: 0 }} className="text-center mb-10">
          <div className="inline-flex items-center gap-2 px-4 py-1.5 rounded-full bg-emerald-500/10 border border-emerald-500/20 text-emerald-400 text-xs font-medium mb-4">
            <BookOpen className="w-3 h-3" />
            Media Literacy Guide
          </div>
          <h1 className="text-3xl md:text-4xl font-bold text-white mb-3">
            How to Spot Fake News
          </h1>
          <p className="text-slate-400 max-w-lg mx-auto text-sm">
            Master the art of critical thinking. These tips will help you identify misinformation and become a more informed reader.
          </p>
        </motion.div>

        {/* Tips Grid */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-10">
          {tips.map((tip, i) => (
            <motion.div
              key={i}
              initial={{ opacity: 0, y: 20 }}
              animate={{ opacity: 1, y: 0 }}
              transition={{ delay: 0.05 * i }}
            >
              <Card className="bg-white/[0.03] border-white/[0.06] hover:bg-white/[0.05] transition-all duration-300 h-full">
                <CardContent className="p-5">
                  <div className={`w-10 h-10 rounded-xl flex items-center justify-center mb-3 ${tip.color}`}>
                    <tip.icon className="w-5 h-5" />
                  </div>
                  <h3 className="text-base font-semibold text-white mb-2">{tip.title}</h3>
                  <p className="text-sm text-slate-400 leading-relaxed">{tip.description}</p>
                </CardContent>
              </Card>
            </motion.div>
          ))}
        </div>

        {/* Quick Checklist */}
        <motion.div
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.3 }}
        >
          <Card className="bg-white/[0.03] border-white/[0.06]">
            <CardContent className="p-6">
              <h2 className="text-lg font-bold text-white mb-5">Quick Credibility Checklist</h2>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
                {quickChecklist.map((item, i) => (
                  <div key={i} className="flex items-start gap-2.5">
                    {item.check ? (
                      <CheckCircle className="w-4 h-4 text-emerald-400 mt-0.5 flex-shrink-0" />
                    ) : (
                      <XCircle className="w-4 h-4 text-red-400 mt-0.5 flex-shrink-0" />
                    )}
                    <span className="text-sm text-slate-300">{item.text}</span>
                  </div>
                ))}
              </div>
            </CardContent>
          </Card>
        </motion.div>
      </div>
    </div>
  );
}

import { clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs) {
  return twMerge(clsx(inputs))
} 


export const isIframe = window.self !== window.top;

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/14bfcfdb-29c6-46ae-b3f9-66ef52757d2b" />

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/a349394f-457e-4265-a326-563c388363bf" />



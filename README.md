import { useState } from "react";
import axios from "axios";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Card, CardContent } from "@/components/ui/card";
import { Select, SelectItem } from "@/components/ui/select";
import { Progress } from "@/components/ui/progress";
import { toast } from "@/components/ui/toast";
import { useSession } from "@/hooks/useSession";
import { saveAs } from "file-saver";

export default function ResumeAnalyzer() {
  const { user, isPremium, login, logout } = useSession();
  const [resumeText, setResumeText] = useState("");
  const [file, setFile] = useState(null);
  const [language, setLanguage] = useState("en");
  const [analysis, setAnalysis] = useState(null);
  const [loading, setLoading] = useState(false);
  const [progress, setProgress] = useState(0);
  const [optimizedResume, setOptimizedResume] = useState(null);
  const [darkMode, setDarkMode] = useState(false);

  const analyzeResume = async () => {
    setLoading(true);
    setProgress(30);
    try {
      const response = await axios.post("http://localhost:5000/analyze", {
        resume: resumeText,
        language: language,
        premium: isPremium,
      });
      setAnalysis(response.data);
      setProgress(100);
      toast({ title: "Analysis completed successfully!" });
    } catch (error) {
      console.error("Error analyzing resume", error);
      toast({ title: "Error analyzing resume", variant: "destructive" });
    }
    setLoading(false);
  };

  const optimizeResume = async () => {
    if (!isPremium) {
      toast({ title: "Upgrade to premium for optimization!", variant: "destructive" });
      return;
    }
    setLoading(true);
    setProgress(30);
    try {
      const response = await axios.post("http://localhost:5000/optimize", {
        resume: resumeText,
        language: language,
      });
      setOptimizedResume(response.data.optimized_resume);
      setProgress(100);
      toast({ title: "Resume optimized successfully!" });
    } catch (error) {
      console.error("Error optimizing resume", error);
      toast({ title: "Error optimizing resume", variant: "destructive" });
    }
    setLoading(false);
  };

  const downloadPDF = () => {
    if (!optimizedResume) return;
    const blob = new Blob([optimizedResume], { type: "application/pdf" });
    saveAs(blob, "optimized_resume.pdf");
  };

  return (
    <div className={`max-w-2xl mx-auto p-6 space-y-4 ${darkMode ? "bg-gray-900 text-white" : "bg-white text-black"}`}>
      <h1 className="text-2xl font-bold">AI Resume Analyzer</h1>
      <Button onClick={() => setDarkMode(!darkMode)}>{darkMode ? "Light Mode" : "Dark Mode"}</Button>
      {!user ? (
        <Button onClick={login}>Login</Button>
      ) : (
        <div className="flex justify-between">
          <p>Welcome, {user.name}!</p>
          <Button onClick={logout}>Logout</Button>
        </div>
      )}
      <Textarea
        placeholder="Paste your resume text here..."
        value={resumeText}
        onChange={(e) => setResumeText(e.target.value)}
        className="w-full"
      />
      <Select value={language} onValueChange={setLanguage}>
        <SelectItem value="en">English</SelectItem>
        <SelectItem value="es">Spanish</SelectItem>
        <SelectItem value="fr">French</SelectItem>
        <SelectItem value="de">German</SelectItem>
      </Select>
      <Button onClick={analyzeResume} disabled={loading}>
        {loading ? "Analyzing..." : "Analyze Resume"}
      </Button>
      <Button onClick={optimizeResume} disabled={loading}>
        {loading ? "Optimizing..." : "Optimize Resume (Premium)"}
      </Button>
      {loading && <Progress value={progress} />}
      {analysis && (
        <Card>
          <CardContent className="p-4">
            <h2 className="text-xl font-bold">Analysis Result</h2>
            <p>Score: {analysis.score}%</p>
            <p>Found Keywords: {analysis.found_keywords.join(", ")}</p>
            <p>Missing Keywords: {analysis.missing_keywords.join(", ")}</p>
            <ul className="list-disc pl-5">
              {analysis.recommendations.map((rec, index) => (
                <li key={index}>{rec}</li>
              ))}
            </ul>
            <h3 className="text-lg font-semibold mt-4">AI Insights</h3>
            <p>{analysis.ai_insights || "No additional insights available."}</p>
          </CardContent>
        </Card>
      )}
      {optimizedResume && (
        <Card>
          <CardContent className="p-4">
            <h2 className="text-xl font-bold">Optimized Resume</h2>
            <Textarea value={optimizedResume} readOnly className="w-full" />
            <Button onClick={downloadPDF} className="mt-2">Download as PDF</Button>
          </CardContent>
        </Card>
      )}
      {!isPremium && (
        <div className="p-4 border rounded-lg bg-yellow-100 text-center">
          <p>Upgrade to premium for enhanced features!</p>
          <Button className="mt-2">Upgrade Now</Button>
        </div>
      )}
    </div>
  );
}

